- 当etcd服务当掉之后，服务不会panic
- ```go
  package storage
  
  import (
  	"context"
  	"fmt"
  	"github.com/apisix/manager-api/internal/conf"
  	"github.com/apisix/manager-api/internal/log"
  	"github.com/apisix/manager-api/internal/utils"
  	"github.com/apisix/manager-api/internal/utils/runtime"
  	"github.com/coreos/etcd/client"
  	"strings"
  	"time"
  )
  
  var (
  	v2Client *client.KeysAPI
  )
  
  
  type EtcdV2Storage struct {
  	client *client.KeysAPI
  	etcdConf *conf.Etcd
  }
  
  
  func InitETCDV2Client(etcdConf *conf.Etcd) error {
  	config := client.Config{
  		Endpoints:               etcdConf.Endpoints,
  		HeaderTimeoutPerRequest: 5 * time.Second,
  		Username:                etcdConf.Username,
  		Password:                etcdConf.Password,
  	}
  
  	cli, err := client.New(config)
  	if err != nil {
  		log.Errorf("init etcd failed: %s", err)
  		return fmt.Errorf("init etcd failed: %s", err)
  	}
  
  	clientKsapi := client.NewKeysAPI(cli)
  	if clientKsapi == nil {
  		log.Errorf("trans etcd v2 client to keyapis failed ")
  		return fmt.Errorf("trans etcd v2 client to keyapis failed")
  	}
  
  	v2Client = &clientKsapi
  
  	utils.AppendToClosers(Close)
  	return nil
  }
  
  func NewETCDV2Storage(etcdConf *conf.Etcd) (*EtcdV2Storage, error) {
  	cli, err := client.New(client.Config{
  		Endpoints:   etcdConf.Endpoints,
  		HeaderTimeoutPerRequest: 5 * time.Second,
  		Username:    etcdConf.Username,
  		Password:    etcdConf.Password,
  	})
  
  	if err != nil {
  		panic(err)
  		log.Errorf("init etcd failed:", err.Error())
  		return nil, fmt.Errorf("init etcd failed: %s", err)
  	}
  
  
  	clientKsapi := client.NewKeysAPI(cli)
  	if clientKsapi == nil {
  		log.Errorf("trans etcd v2 client to keyapis failed ")
  		return nil,fmt.Errorf("trans etcd v2 client to keyapis failed")
  	}
  
  
  	s := &EtcdV2Storage{
  		client: &clientKsapi,
  		etcdConf: etcdConf,
  	}
  
  	utils.AppendToClosers(s.Close)
  	return s, nil
  }
  
  
  func (s *EtcdV2Storage) Close() error {
  	// TODO close this etcdv2 connection
  	return nil
  }
  
  func (s *EtcdV2Storage) Get(ctx context.Context, key string) (string, error) {
  	resp, err := (*s.client).Get(ctx,key,&client.GetOptions{
  		Recursive: false,
  		Sort:      true,
  	})
  	if err != nil {
  		log.Errorf("etcd get failed: %s", err)
  		return "", fmt.Errorf("etcd get failed: %s", err)
  	}
  
  	//resp.Node.Nodes should just be only one
  	for _, n := range resp.Node.Nodes {
  
  		if strings.Contains(n.Key, "___lock") {
  			continue
  		}
  
  		return n.Value, nil
  	}
  
  	return "",fmt.Errorf("get %s key failed ", key)
  
  
  }
  
  func (s *EtcdV2Storage) List(ctx context.Context, key string) ([]Keypair, error) {
  	resp, err := (*s.client).Get(ctx,key,&client.GetOptions{
  		Recursive: false,
  		Sort:      true,
  	})
  
  	if err != nil {
  		log.Errorf("etcd get failed: %s", err)
  		return nil, fmt.Errorf("etcd get failed: %s", err)
  	}
  
  	var ret []Keypair
  
  	for _,node := range resp.Node.Nodes {
  
  		if strings.Contains(node.Key, "___lock") {
  			continue
  		}
  
  		data := Keypair{
  			Key:   node.Key,
  			Value: node.Value,
  		}
  
  		ret = append(ret, data)
  	}
  
  	return ret, nil
  }
  
  func (s *EtcdV2Storage) Create(ctx context.Context, key, val string) error {
  	_, err := (*s.client).Create(ctx, key, val)
  	if err != nil {
  		log.Errorf("etcd put failed: %s", err)
  		return fmt.Errorf("etcd put failed: %s", err)
  	}
  	return nil
  }
  
  func (s *EtcdV2Storage) Update(ctx context.Context, key, val string) error {
  	_, err := (*s.client).Update(ctx, key, val)
  	if err != nil {
  		log.Errorf("etcd put failed: %s", err)
  		return fmt.Errorf("etcd put failed: %s", err)
  	}
  	return nil
  }
  
  func (s *EtcdV2Storage) Delete(ctx context.Context,key string) error{
  
  	_,err := (*s.client).Delete(ctx, key,&client.DeleteOptions{Dir: false})
  
  	if err != nil{
  		log.Errorf("delete etcd key[%s] failed: %s",key,err)
  		return fmt.Errorf("delete etcd key[%s] failed: %s",key,err)
  	}
  	return nil
  }
  
  
  func (s *EtcdV2Storage) BatchDelete(ctx context.Context, keys []string) error {
  	for i := range keys {
  		_,err := (*s.client).Delete(ctx, keys[i],&client.DeleteOptions{Dir: false})
  		if err != nil {
  			log.Errorf("delete etcd key[%s] failed: %s", keys[i], err)
  			return fmt.Errorf("delete etcd key[%s] failed: %s", keys[i], err)
  		}
  	}
  	return nil
  }
  
  func (s *EtcdV2Storage) Watch(ctx context.Context, key string) <-chan WatchResponse {
  	watcher := (*s.client).Watcher(key,&client.WatcherOptions{AfterIndex: 0,Recursive:true})
  	ch := make(chan WatchResponse, 1)
  
  	go func(){
  		defer runtime.HandlePanic()
  		for{
  			response,err := watcher.Next(ctx)
  			output := WatchResponse{
  				Canceled: false,
  			}
  			if err != nil || response == nil {
  				InitETCDV2Client(s.etcdConf)
  				if strings.Contains(err.Error(),"connection refused") {
  					watcher = (*s.client).Watcher(key,&client.WatcherOptions{AfterIndex: 0,Recursive:true})
  				}
  				continue
  			}
  
  			if !response.Node.Dir {
  
  				e := Event{
  					Keypair: Keypair{
  						Key:   response.Node.Key,
  						Value: response.Node.Value,
  					},
  				}
  
  				switch response.Action {
  				case "set":
  					e.Type = EventTypePut
  				case "delete":
  					e.Type = EventTypeDelete
  				}
  
  				output.Events = append(output.Events,e)
  
  			}
  			if output.Canceled {
  				log.Error("channel canceled")
  				output.Error = fmt.Errorf("channel canceled")
  			}
  			ch <- output
  	  }
  	  close(ch)
  	}()
  
  	return ch
  }
  ```