- ```yaml
  GIT_SHA=$(shell git rev-parse HEAD)
  
  
  ## help:             Show Makefile rules.
  .PHONY: help
  help: default
  	@echo Makefile rules:
  	@echo
  	@grep -E '^## [-A-Za-z0-9_]+:' Makefile | sed 's/##/   /'
  
  
  ## init: make same init work
  .PHONY: init
  init:
  	@commitizen init cz-conventional-changelog --save --save-exact
  
  ## changelog: generate changelog
  .PHONY: changelog
  changelog:
  	@conventional-changelog -p angular -i CHANGELOG.md -w -r 0 > CHANGELOG.md
  
  ## commit: git commit
  .PHONY: commit
  commit:
  	@git cz
  
  
  ## image: make image
  .PHONY: image
  image:
  	@echo registry.cn-hangzhou.aliyuncs.com/leo_apps/bdgateway-v2:${GIT_SHA}
  	@docker build -t registry.cn-hangzhou.aliyuncs.com/leo_apps/bdgateway-v2:${GIT_SHA} .
  
  ## deploy: make deploy
  .PHONY: deploy
  deploy:
  	@echo registry.cn-hangzhou.aliyuncs.com/leo_apps/bdgateway-v2:${GIT_SHA}
  	@docker push registry.cn-hangzhou.aliyuncs.com/leo_apps/bdgateway-v2:${GIT_SHA}
  
  ```
- ## Books
	- ![跟我一起写makefile.pdf](../assets/跟我一起写makefile_1652341617662_0.pdf) #book
	-
	-