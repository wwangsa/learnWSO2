# [Udemy - Learn WSO2 MI: a Step Guide to Master ESB & API Integration](https://www.udemy.com/course/learn-wso2-mi-a-step-guide-to-master-esb-api-integration/)
Instructor: Nelson Dias
Github: https://github.com/nelsonandredias/wso2_course


## Tools
 * [WSO Integration Studio](https://wso2.com/integration/integration-studio/)

## Deploying code

To pull this code and save it in default workspace (using WSO2 IS v8.0.0)
```shell
cd ~/IntegrationStudio/8.0.0/workspace
git clone https://github.com/wwangsa/learnWSO2.git
```

## Section 1: Introduction

6. Create the project Zero (Project folder: ProjectZero)
	Create an endpoint where name is passed as path param and it will return a greeting message "Welcome {name}".

	```shell
		curl -v GET "http://localhost:8290/greetings/Nelson" -w "\n"
	```

	* Resource: Define the api path
	* Property: It is used to manipulate properties. A formula can be used to concatenate the values from query strings or path params
	* Payload: Set how the output looks like based on message created by property
	

## Section 2: Message entry points

12. Intro
	```shell
		curl -v GET "http://localhost:8290/" -w "\n"
	```
	When no resource specified, the curl will return 404 while the log mediator will report "main sequence executed for call to non-existent". This message comes can be customizedfrom
	{IntegrationStudio_Home}/runtime/microesb/repository/deployment/server/synapse-configs/default/sequences/main.xml and add the following underneath the first custom log
	```xml
	<log descripton="LOG CUSTOM MAIN SEQUENCE" level="custom">
            <property name="LOG CUSTOM DEFAULT MESSAGE" value="this messages is part of the default main sequence"/>
    </log>

	```
	With the changes above, when we run the curl without resource specified, we will get 405 Methods not allowed when doing curl. While the logmediator will report the xml message above.

13. REST APIs - Default API Resource (Project folder: ECommerceAPI)
	Using Log mediator to generate log message.

	```shell
		curl -v GET "http://localhost:8290/orders" -w "\n"
	```

	```shell
	#On Micro Integrator Server Log, it generates the following line when the curl is executed
	2022-02-09 23:36:51,656]  INFO {LogMediator} - {api:EcommerceAPI} LOG MESSAGE = this is the default resource
	```

## Section 3: Message processing units


## Section 4: Message exit points


## Section 5: Data Services


