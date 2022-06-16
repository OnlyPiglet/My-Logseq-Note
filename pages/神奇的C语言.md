- ### 结构体
	- ```c
	  typedef struct {
	      void *_PlaceHolder;
	  }Person;
	  
	  
	  typedef struct {
	      union {
	          Person *person;
	          char const *name;
	      };
	      int age;
	      char intro[];
	  }InternalPerson;
	  ```