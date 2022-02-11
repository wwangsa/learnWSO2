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

## Tips
* When doing google search, use the following keywords. Most of the time, google search will point to old article so specifying version would help narrow down the latest result. WSO2 Enterprise Integrator (EI) was introduced on v6.
	* ESB: WSO2 EI ESB {version}
	* Data service server: WSO2 EI DSS {version}
* When dragging and drop mediators, sometimes the mediators keep dragging the previous mediator. This is the IDE bug. You have to click the new mediator until it got highlighted, then you drag it.
* To stop the console run, there is a maximize icon on top right of the console window. When it's click, you will see the stop button

## References
* To learn more different types of mediators and what it could do, here is the [link to the v 7.2.0](https://ei.docs.wso2.com/en/7.2.0/micro-integrator/references/mediators/about-mediators/#!) with Micro Integrator.

## Section 1: Introduction

6. Create the project Zero (Project folder: ProjectZero)
	Create an endpoint where name is passed as path param and it will return a greeting message "Welcome {name}".

	Call GET greetings/{name}
	```shell
		curl -v GET "http://localhost:8290/greetings/Nelson" -w "\n"
	```
	
	Note on Mediators:
	1. API: Where the it defines the part of url. In the ex above, ie. greetings
	2. Resource: Where the endpoints path are defined. In the ex above, ie. name
	3. Log: To generate message in the log file
	4. Property: It is used to manipulate messages. A formula can be used to concatenate the values from query strings or path params
	5. Payload Factory: Transforms or replaces the contents of a message in other words set how the output looks like based on message created by property
	6. Inflow and Outflow Mediators: inflow refers to request sequence while outflow refers to response sequence. Each sequence has childs in it such as log, property, payload factory and so on.
	7. To run the project, right click on the "ProjectZeroCompositeExporter" and "Export Projects Artifact and Run"
	

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
		# GET orders
		curl -v GET "http://localhost:8290/orders" -w "\n"
	```

	```shell
	# On Micro Integrator Server Log, it generates the following line when the curl is executed
	2022-02-09 23:36:51,656]  INFO {LogMediator} - {api:EcommerceAPI} LOG MESSAGE = this is the default resource
	```

14. REST APIs - URI Template

	Resource's URL Style: URI_TEMPLATE
	
	Allowing path on the resource url

	Message flows: 
	1. On the new API Resource, define the URI_TEMPLATE so we can use the path params (`$ctx:uri.var.currentMonth`) and query strings (`$ctx:query.param.minday` and `$ctx:query.param.maxday`). 
	2. Log Message is used to log and capture the path params and query strings then hold them into variables.
	3. Set Payload is used to define the message output based on variables passed onto the output
	4. Loop Back mediator is used to move the message from inflow (request path) to outflow (response path) sequence.  All the configuration included in the in sequence that appears after the Loopback mediator is skipped.
	5. Respond mediator is used to send back the result back to the client

	Call `GET orders/month/{currentMonth}`
	```shell
		#Before adding the query strings in the log message
		curl -v GET "http://localhost:8290/orders/month/January" -w "\n"
	```
	Result `GET orders/month/{currentMonth}`
	```json
		{
        	"month":January
		}
	```
	Call `GET orders/month/{currentMonth}?*`
	```shell
		#After adding the query strings in the log message
		curl -v GET "http://localhost:8290/orders/month/January?minday=1&maxday=20" -w "\n"
	```
	Result `GET orders/month/{currentMonth}?*`
	```json
		{
			"month":January,
			"minday":1,
			"max":20,
			"totalOrders": 200
		}
	```

15. REST APIs - URL Mapping
	
	Resource's URL style - URI_MAPPING
	
	Request that match the mapping pattern will be processed. There are 3 kinds of url mapping:
	* Extension mapping. Ex: `/.jsp`
	* Exact mapping. Ex: `/test` 
	* Path mapping. Ex: `/test/*`
	
	Be careful when constructing JSON. Missing double quote will compiled ok but would not show result

	Call `GET orders/list`
	```shell
		#After adding the query strings in the log message
		curl -v GET "http://localhost:8290/orders/list" -w "\n"
	```
	Result `GET orders/month/{currentMonth}?*`
	```json
		{
			"orders":{
				"order":{
					"country":"Portugal",
					"total":394
				},
				"order":{
					"country":"Spain",
					"total":212
				},
				"order":{
					"country":"Brazil",
					"total":690
				}
			}
		}

	```


## Section 3: Message processing units


## Section 4: Message exit points


## Section 5: Data Services


