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
* When dragging and drop mediators, sometimes the mediators keep dragging the previous mediator. This is the IDE bug. You have to click the new mediator until it got highlighted (may have to click multiple times), then you drag it.
* To stop the console run, there is a maximize icon on top right of the console window. When it's click, you will see the stop button

## References
* To learn more different types of mediators and what it could do, here is the [link to the v 7.2.0](https://ei.docs.wso2.com/en/7.2.0/micro-integrator/references/mediators/about-mediators/#!) with Micro Integrator.

## Section 1: Introduction

6. Create the project Zero (Project folder: ProjectZero)
	* When creating new integration project, use the "New Integration Project". This will create a folder that contains with *ESBConfigs* and *CompositeExporter* folders undeneath it.
	* To add REST API Project, click on the *ESBConfigs* folder > New> REST API > Create A New API Artifact

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

	```log
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

16. REST APIs - Swagger files (Project folder: SwaggerFile)
	
	* After the project is created and project artifacts exported, the Swagger Editor tab will shows up next to Design and Source tabs for the xml file you edit. This is due to built in Swagger processor in WSO2. 
	* The processors setting can be found in {Integration_Home}/runtime/microesb/conf/carbon.xml. 
	* To access the project's swagger, go to either urls: 
		* http://localhost:8290/PetsAPI?swagger.json 
		* http://localhost:8290/PetsAPI?swagger.yaml (download)
	* We can download the yaml file and upload it to the Postman so we don't have to set it up in Postman manually. One thing to note that Postman will use https url.

	Below is to test manually using curl
	Call `GET api/pets`
	```shell
		#After adding the query strings in the log message
		curl -v GET "http://localhost:8290/api/pets" -w "\n"
	```
	Result `GET api/pets`
	```json
		{
			"pet":{
				"name":"Kuky",
				"animal":"Dog"
			},
			"pet":{
				"name":"Kuky",
				"animal":"Cat"
			},
			"pet":{
				"name":"Star",
				"animal":"Cat"
			},
			"pet":{
				"name":"Gems",
				"animal":"Mouse"
			}
		}
	```

	Call `GET api/pets/{name}`
	```shell
		#After adding the query strings in the log message
		curl -v GET "http://localhost:8290/api/pets/Kuky" -w "\n"
	```
	Result `GET api/pets{name}`
	```json
		{
			"pet":{
				"name":"Kuky",
				"animal":"Dog"
			},
			"pet":{
				"name":"Kuky",
				"animal":"Cat"
			}
		}
	```

17. Proxy services (Project folder: ProxyService)

	The instructor use a SOAP service for the Proxy services example. The proxy services used to perform transformation and introduce functionality without changing the existing service. He provided 2 files:
	1. Java based SOAP service. The webservice can be accessed at http://localhost:9000/services/SimpleStockQuoteService

	```shell
		#Prerequisites. The instructor uses Java 1.8 or Java 8 (OpenJDK). It would not work with 11 or higher. Run the following to install java 8.
		sudo apt install openjdk-8-jdk
		#create symbolic link so we can execute it by calling java8
		sudo ln -s /usr/lib/jvm/java-8-openjdk-amd64/bin/java /usr/bin/java8

		#To run the web services, execute this with java 8.   
		java8 -jar Resources/Chapter_17_Proxy_Services/stockquote_service.jar
	```	
	2. Import WSDL file using Postman, SoapUI or Curl

		The instructor is using SoapUI. Below is the instruction to use Postman or curl

		a. Under Postman's colection, expand SimpleStockQuoteService > SimpleStockQuoteServiceHttpSoap12Endpoint/getQuote then go to the Body and change the value under the symbol tag to `IBM`. 


		b. Using Curl
		* Using xml file as data reference. Due to relative path, make sure to run it on the workspace folder.
		```shell
			curl --location --request POST 'http://localhost:9000/services/SimpleStockQuoteService' \
			--header 'Content-Type: text/xml; charset=utf-8' \
			--header 'SOAPAction: urn:getQuote' \
			--data @Resources/Chapter_17_Proxy_Services/requestBody.xml
		```
		* Without using xml file
		```shell
			curl --location --request POST 'http://localhost:9000/services/SimpleStockQuoteService' \
			--header 'Content-Type: text/xml; charset=utf-8' \
			--header 'SOAPAction: urn:getQuote' \
			--data-raw '<?xml version="1.0" encoding="utf-8"?>
			<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
			<soap:Body>
				<getQuote xmlns="http://services.samples">
				<request>
					<symbol>string</symbol>
				</request>
				</getQuote>
			</soap:Body>
			</soap:Envelope>
			'
		```

	Once the test successful, we are creating the proxy services where we register the WSDL file into the Registry Resources

	1. Create new integration project called `ProxyService` on the workspace folder but this time 3 Modules need to be created by checking the checkboxes for: `ESB Configs`, `Composite Exporter`, and `Registry Resources`. 

	2. On `ProxyServiceConfigs`, 
		* create new > `Proxy Services` called `CustomProxyServiceStockQuoteService` 

	3. On `ProxyServiceRegistryResources`, 
		* create new > Registry Resource. Click next the import from the file system.
		* Click on browse file then locate the wsdl file in the Resources/Chapter_17_Proxy_Services/sample_proxy_1.wsdl 

	4. Go back to the `CustomProxyServiceStockQuoteService` and click on its property 
		* scroll down to the WSDL Type and select `REGISTRY_KEY`
		* Click WSDL Key and select  *workspace > Carbon Application Registry Resources > ProxyServiceRegistryResources > sample_proxy_1_wsdl > `/_system/governance/custom/sample_proxy_1.wsdl`*

	5. Now create the proxy service flow
		* Add Log Start mediator
		* Add Send mediator (allowing sending request to the backend)
		* Go back `src/main/synapse-config/endpoints`, 
			* create new `Endpoint` called `SimpleStockQuote` 
			* add the address `http://localhost:9000/services/SimpleStockQuoteService` then click Finish
			* Change the Address Endpoint Format to `SOAP 1.2`
		* Go back to the `CustomProxyServiceStockQuoteService` and add the Defined Endpoints > `SimpleStockQuote` next to the Send mediator
		* Add Log END mediator in the output flow and add another Send mediator

	6. To run the Proxy Service, we need to package by right click on `ProxyServiceCompositeExporter` then *Export Project artifacts and run* then select the following then click Finish
		* ProxyServiceRegistryResources
		* ProxyServiceConfigs

	7. Make sure the stockquote_service.jar is running
		```shell
			#To run the web services, execute this with java 8.   
			java8 -jar Resources/Chapter_17_Proxy_Services/stockquote_service.jar
		```
	8. Once exported, the WSDL file can be viewed and downloaded from http://localhost:8290/services/CustomProxyServiceStockQuoteService?wsdl into either POSTMAN, SOAP UI or curl
		* For SOAP UI
			* Create SOAP Project
				* Project Name: `CustomProxyService`
				* Initial WSDL: `http://localhost:8290/services/CustomProxyServiceStockQuoteService?wsdl`
			* Go to any CustomProxyBinding, edit the getQuote and replace question mark in the `<xsd:symbol>?</xsd:symbol>` with either `IBM` or `MSFT`. It should return the value while the WSO2 will logs the following
			```log			
				[2022-02-16 00:51:32,795]  INFO {LogMediator} - {proxy:CustomProxyServiceStockQuoteService} LOG MESSAGE = LOG START
				[2022-02-16 00:51:32,904]  INFO {TimeoutHandler} - This engine will expire all callbacks after GLOBAL_TIMEOUT: 120 seconds, irrespective of the timeout action, after the specified or optional timeout
				[2022-02-16 00:51:34,395]  INFO {LogMediator} - {proxy:CustomProxyServiceStockQuoteService} LOG MESSAGE = LOG END

			```


18. Inbound Endpoints - Listening Inbound Endpoints
	
	An inbound endpoint is a message entry point that can inject messages directly from the transport layer to the mediation layer, without going through the Axis engine.  One of the advantages of using Inbound Endpoints is in its ability to create inbound messaging channels dynamically. There are three types of inbound endpoints:
	1. Listening Inbound Endpoints (Bidirectional) - HTTP/HTTPS, HL7, CXF WS-RM or Websocket
	2. Polling Inbound Endpoints (One directional) - File, JMS or Kafka
	3. Event-based Inbound Endpoints (Pulled once connection established) - MQTT or RabbitMQ

	Note: Port 8285, 8290 and 8295 are being used. Make sure the ports are not taken already. Otherwise, it will give error.

	Steps (mixing design and code view for quicker note):
	1. Create Integration Project called `InboundEndpoint` with ESB Configs and CompositeExporter
	2. Create RestAPI project called `DictionaryAPI` with context `/api`
		
		a. Set Resource properties -> URI Style: `URI_Template`, Uri Template: `/dictionary/{word}`, Protocol: `http`

		b. InSequence - Set Log Description
		```xml
			<log description="LOG MESSAGE" level="custom">
					<property expression="$ctx:uri.var.word" name="LOG MESSAGE"/>
			</log>
		```

		c. InSequence - Set property from the log word mediator
		```xml
			<property description="SET PROPERTY" expression="$ctx:uri.var.word" name="wordProperty" scope="default" type="STRING"/>
		```

		d. InSequence - Set payloadFactory that display the word Property
		```xml
			<payloadFactory description="SET PAYLOAD" media-type="json">
					<format>
						{
							"word":$1
						}
					</format>
					<args>
						<arg evaluator="xml" expression="$ctx:wordProperty"/>
					</args>
				</payloadFactory>
		```

		e. InSequence - Add Loopback to move to OutSequence
		```xml
			<loopback/>
		```

		f. OutSequence - Add Respond back to the client
		```xml
			<respond description="SEND OUT RESPONSE"/>
		```

	3. Create inbound endpoints called `ExposeRestApiIEP`

		a. On the Inbound EP, set the Dispatch Filter Pattern to `/api/dictionary/.*`. The `.*` means it only work as filter dispatcher and it will ignore the log inside the sequence. Then change the Inbound Http Port: `8285` 

		b. Add Sequence to sequence box. This create the following
		```xml
			<sequence name="Sequence" trace="disable" xmlns="http://ws.apache.org/ns/synapse"/>			
		```

		c. Double clicking sequence mediator and add log mediator in it. This will create `Sequence.xml` file
		```xml
			<log description="LOG MESSAGE" level="custom">
        		<property name="LOG MESSAGE" value="THIS SEQUENCE HAS BEEN EXECUTED"/>
    		</log>
		```
	
	4. Create inbound endpoints called `HttpTestIEP` without dispatcher. This means sequence will actually be called.
		
		a. On the Inbound EP, set the Port to `8295`  
	
	5. Crete an endpoint called `DictionaryApiEP` 
	
		a. Endpoint Type: `HTTP Endpoint`
		
		b. URI Template: `http://localhost:8290/api/dictionary/{uri.var.wordIEP}`


	6. Create another sequence called `SequenceIN`
		
		a. Drag Log mediator inside the sequence box
		```xml
			<log description="LOG MESSAGE" level="custom">
        		<property name="LOG MESSAGE" value="SEQUENCE IN HAS BEEN EXECUTED"/>
    		</log>
		```
		b. Add property mediator after that
		```xml
			<property description="SET PROPERTY" name="uri.var.wordIEP" scope="default" type="STRING" value="covid"/>
		```

		c. Add Send mediator then add  `DictionaryApiEP` defined endpoints
		```xml
			<send>
				<endpoint key="DictionaryApiEP"/>
			</send>
		```
	
	7. Create another sequence to end the loop. It's called `SequenceOUT`

		a. Add log mediator
		```xml
			<log description="LOG MESSAGE" level="custom">
        		<property name="LOG MESSAGE" value="SEQUENCE OUT HAS BEEN EXECUTED"/>
    		</log>
		```

		b. Add send mediator inside the sequence box
		```xml
			<send/>
		```

	8. Back to the `HttpTestIEP.xml` inbound-endpoints

		a. Drag `SequenceIN` from Defined Sequences into sequence


	9. Back to `SequenceIN.xml` 

		a. change the Receiving Sequence Type: `Static`

		b. set SequenceOUT directly on the code
		```xml
			<send receive="SequenceOUT">
        		<endpoint key="DictionaryApiEP"/>
    		</send>
		```
	10. Package artificats of the first draft by going to Exporter `Export Artificats Project and Run`. The application is successfully deployed when you see the listeners are ready in the console

	```log
		[2022-02-18 00:13:20,149]  INFO {HTTPEndpointManager} - Listener is already started for port : 8295
		[2022-02-18 00:13:20,150]  INFO {HTTPEndpointManager} - Listener is already started for port : 8285
	```

	11. Open terminal window to test out
		
		a. Calling  `DictionaryAPI` REST API project
		```shell
			curl -v GET "http://localhost:8290/api/dictionary/Portugal"
		```
		Response
		```json
			{
				"word":Portugal
			}
		```
		Log Message
		```log
			[2022-02-18 00:18:29,431]  INFO {LogMediator} - {api:DictionaryAPI} LOG MESSAGE = Portugal
		```
		
		b. Calling `HttpTestIEP` Inbound endpoint - through dispatch filter pattern exposing the API via http endpoint
		```shell
			curl -v GET "http://localhost:8285/api/dictionary/wso2" -w "\n"
		```
		Response
		```json
			{
				"word":wso2
			}
		```
		Log Message
		```log
			[2022-02-18 00:21:42,147]  INFO {LogMediator} - {api:DictionaryAPI} LOG MESSAGE = wso2
		```

		c. Calling `HttpTestIEP` Inbound endpoint - Since there is no dispatch filter, the sequencesIN is used
		```shell
			curl -v GET "http://localhost:8295" -w "\n"
		```
		Response
		```json
			{
				"word":covid
			}
		```
		Log Message
		```log
			[2022-02-18 00:23:57,526]  INFO {LogMediator} - {inboundendpoint:HttpTestIEP} LOG MESSAGE = SEQUENCE IN HAS BEEN EXECUTED
			[2022-02-18 00:23:57,552]  INFO {TimeoutHandler} - This engine will expire all callbacks after GLOBAL_TIMEOUT: 120 seconds, irrespective of the timeout action, after the specified or optional timeout
			[2022-02-18 00:23:57,629]  INFO {LogMediator} - {api:DictionaryAPI} LOG MESSAGE = covid
			[2022-02-18 00:23:57,697]  INFO {LogMediator} - {inboundendpoint:HttpTestIEP} LOG MESSAGE = SEQUENCE OUT HAS BEEN EXECUTED
		```


## Section 3: Message processing units


## Section 4: Message exit points


## Section 5: Data Services


