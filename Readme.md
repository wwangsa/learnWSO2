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

![Big Picture for Ch 6, 12-17](Resources/screenshots/chbasics.png)

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
		* create new > `Proxy Services` artifact called `CustomProxyServiceStockQuoteService` 

	3. On `ProxyServiceRegistryResources`, 
		* create new > Registry Resource . Click next the import from the file system.
		* Click on browse file then locate the wsdl file in the Resources/Chapter_17_Proxy_Services/sample_proxy_1.wsdl 

	4. Go back to the `CustomProxyServiceStockQuoteService` and click on its property 
		* scroll down to the WSDL Type and select `REGISTRY_KEY`
		* Click WSDL Key and select  *workspace > Carbon Application Registry Resources > ProxyServiceRegistryResources > sample_proxy_1_wsdl > `/_system/governance/custom/sample_proxy_1.wsdl`*

	5. Now create the proxy service flow
		* Add Log Start mediator
		* Add Send mediator (allowing sending request to the backend)
		* Go back `src/main/synapse-config/endpoints`, 
			* create new `Endpoint` artifact called `SimpleStockQuote` 
			* add the address `http://localhost:9000/services/SimpleStockQuoteService` then click Finish
			* Change the Address Endpoint Format to `SOAP 1.2`
		* Go back to the `CustomProxyServiceStockQuoteService` artifact and add the Defined Endpoints > `SimpleStockQuote` next to the Send mediator
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


18. Inbound Endpoints - Listening Inbound Endpoints (Project folder: InboundEndpoint)

![Big Picture for Ch 18](Resources/screenshots/ch18.png)	
	An inbound endpoint is a message entry point that can inject messages directly from the transport layer to the mediation layer, without going through the Axis engine.  One of the advantages of using Inbound Endpoints is in its ability to create inbound messaging channels dynamically. There are three types of inbound endpoints:
	1. Listening Inbound Endpoints (Bidirectional) - HTTP/HTTPS, HL7, CXF WS-RM or Websocket
	2. Polling Inbound Endpoints (One directional) - File, JMS or Kafka
	3. Event-based Inbound Endpoints (Pulled once connection established) - MQTT or RabbitMQ

	Note: 
	+ Port 8285, 8290 and 8295 are being used on this project. Make sure the ports are not taken already. Otherwise, it will give error.
	+ This projects used a lot of sequence artifacts and mediation sequences / sequence mediator. The sequence mediator contains mediators that would be use repeatedly. Think of it as desigining a class and its methods in object oriented.

	Steps (mixing design and code view for quicker note):
	1. Create Integration Project called `InboundEndpoint` with ESB Configs and CompositeExporter
	2. Create RestAPI artifact called `DictionaryAPI` with context `/api`. We're creating `http://localhost:8290/api/dictionary/{word}`endpoint.
		
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

	3. Create inbound endpoints artifact called `ExposeRestApiIEP`. We are creating `http://localhost:8285/api/dictionary/{word}` endpoint that forwards the `{word}` query path to `http://localhost:8290/api/dictionary/{word}`

		a. On the Inbound EP, 
			
		1. set the Dispatch Filter Pattern to `/api/dictionary/.*`. The `.*` means it only work as filter dispatcher and it will ignore the log inside the sequence and forward the response.			
		2. Then change the Inbound Http Port: `8285` 

		b. Since we could not add log and other mediators in the Inbound Endpoint, we have to add sequence. Drag the sequence mediator to the sequence box and double click it. This create `Sequence.xml` file under sequences folder. The codes behind the sequence.xml is the following
		```xml
			<sequence name="Sequence" trace="disable" xmlns="http://ws.apache.org/ns/synapse"/>			
		```

		c. Double clicking sequence mediator and add log mediator in it. This will create `Sequence.xml` file
		```xml
			<log description="LOG MESSAGE" level="custom">
        		<property name="LOG MESSAGE" value="THIS SEQUENCE HAS BEEN EXECUTED"/>
    		</log>
		```
	
	4. Create inbound endpoints artifact called `HttpTestIEP` without dispatcher. This means sequence will actually be called and transformed the message. We are creating `http://localhost:8295` endpoint that will generate `covid` result by passing the query path to`http://localhost:8290/api/dictionary/covid`
		
		a. On the Inbound EP, set the Port to `8295`  
	
		b. Create an endpoint artifact called `DictionaryApiEP` 
		
		1. Endpoint Type: `HTTP Endpoint`			

		2. URI Template: `http://localhost:8290/api/dictionary/{uri.var.wordIEP}`

		c. Create another sequence artifact called `SequenceIN`
			
		1. Drag Log mediator inside the sequence box
		```xml
			<log description="LOG MESSAGE" level="custom">
				<property name="LOG MESSAGE" value="SEQUENCE IN HAS BEEN EXECUTED"/>
			</log>
		```
		2. Add property mediator after that
		```xml
			<property description="SET PROPERTY" name="uri.var.wordIEP" scope="default" type="STRING" value="covid"/>
		```

		3. Add Send mediator then add  `DictionaryApiEP` defined endpoints
		```xml
			<send>
				<endpoint key="DictionaryApiEP"/>
			</send>
		```
	
		d. Create another sequence artifact to end the loop. It's called `SequenceOUT`

		1. Add log mediator
		```xml
			<log description="LOG MESSAGE" level="custom">
				<property name="LOG MESSAGE" value="SEQUENCE OUT HAS BEEN EXECUTED"/>
			</log>
		```

		2. Add send mediator inside the sequence box
		```xml
			<send/>
		```

		e. Back to the `HttpTestIEP.xml` inbound-endpoints

		1. Drag `SequenceIN` from Defined Sequences into sequence

		f. Back to `SequenceIN.xml` 

		1. change the Receiving Sequence Type: `Static`

		2. set SequenceOUT directly on the code
			```xml
				<send receive="SequenceOUT">
					<endpoint key="DictionaryApiEP"/>
				</send>
			```
	5. Package artifacts of the first draft by going to Exporter `Export Artificats Project and Run`. The application is successfully deployed when you see the listeners are ready in the console

	```log
		[2022-02-18 00:13:20,149]  INFO {HTTPEndpointManager} - Listener is already started for port : 8295
		[2022-02-18 00:13:20,150]  INFO {HTTPEndpointManager} - Listener is already started for port : 8285
	```

	6. Open terminal window to test out
		
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
		
		b. Calling `ExposeRestApiIEP` Inbound endpoint - through dispatch filter pattern exposing the API via http endpoint.Since we're using dispatch filter pattern, the sequence is being ignored.
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

19. Inbound Endpoints - Polling Inbound Endpoints (Project folder: InboundEndpoint)

	A polling inbound endpoint polls periodically for data and when data is available the data is injected to a given sequence. Continuing from previous project, this chapter will check when different file types are dropped to the certain folders and it will get copied over to different folder.

	1. Create folders structures for this project
	
	```shell
		mkdir -p /tmp/WSO2/Ch19/CSV/{In,Original,Out}
		mkdir -p /tmp/WSO2/Ch19/JSON/{In,Original,Out}
		mkdir -p /tmp/WSO2/Ch19/XML/{In,Original,Out}
		cp ~/IntegrationStudio/8.0.0/workspace/Resources/Ch_19_Inbound_Endpoints_Polling/sample.csv /tmp/WSO2/Ch19/CSV/
		cp ~/IntegrationStudio/8.0.0/workspace/Resources/Ch_19_Inbound_Endpoints_Polling/sample.json /tmp/WSO2/Ch19/JSON/
		cp ~/IntegrationStudio/8.0.0/workspace/Resources/Ch_19_Inbound_Endpoints_Polling/test.xml /tmp/WSO2/Ch19/XML/
		cd ~/IntegrationStudio/8.0.0/workspace/
	```

	2. Create new sequence artifacts called
		
		a. `CSVDataFileProcessSeq`
		
		* Drag Log to the sequence box
			```xml
				<log description="LOG CSV FILE CONTENT" level="full"/>
			```
		b. `XMLDataFileProcessSeq`

		* Drag Log mediator to the sequence box
			```xml
				<log description="LOG XML FILE CONTENT" level="full"/>
			```

		* Drag another Log mediator to the sequence box. This log uses expression to read the country data inside the xml file.
			```xml
				<log description="LOG MESSAGE" level="custom">
					<property expression="$body//orders//order//country" name="LOG COUNTRY"/>
				</log>
			```

		c. Create new sequence artifact called `JSONDataFileProcessSeq`

		* Drag Log mediator to the sequence box
			```xml
				<log description="LOG FILE JSON CONTENT" level="full"/>
			```

		* Drag another Log mediator to the sequence sbox. This log uses expression to read the country data inside the xml file.
			```xml
				<log description="LOG MESSAGE" level="custom">
					<property expression="json-eval($.id)" name="LOG ID"/>
				</log>
			```

		d. Create new sequence artifact called `ErrorProcessSeq`. 

		This will monitor the error during the 3 sequences above

		* Drag Log mediator to the sequence box
			```xml
				<log description="LOG ERROR DETAILS" level="full">
					<property name="MESSAGE" value="An unexpected error occured!"/>
					<property expression="$ctx:ERROR_CODE" name="ERROR_CODE"/>
					<property expression="$ctx:ERROR_MESSAGE" name="ERROR_MESSAGE"/>
					<property expression="$ctx:ERROR_DETAIL" name="ERROR_DETAIL"/>
					<property expression="$ctx:ERROR_EXCEPTION" name="ERROR_EXCEPTION"/>
				</log>
			```

	3. Create 3 different inbound endpoints artifacts called

		a. `CSVDataFileProcessIEP`
		
		* Inbound Endpoint Creation Type: `File`
		* Sequence: `CSVDataFileProcessSeq`
		* Error Sequence: `ErrorProcessSeq` 
		
		Click Finish
		
		* Change InboundEP Mediator Properties:		
			```xml
				<parameters>
					<parameter name="interval">1000</parameter>
					<parameter name="sequential">true</parameter>
					<parameter name="coordination">true</parameter>
					<parameter name="transport.vfs.ContentType">text/plain</parameter>
					<parameter name="transport.vfs.LockReleaseSameNode">false</parameter>
					<parameter name="transport.vfs.AutoLockRelease">false</parameter>
					<parameter name="transport.vfs.ActionAfterFailure">MOVE</parameter>
					<parameter name="transport.vfs.FailedRecordsFileName">vfs-move-failed-records.properties</parameter>
					<parameter name="transport.vfs.FailedRecordsFileDestination">repository/conf/</parameter>
					<parameter name="transport.vfs.MoveFailedRecordTimestampFormat">dd-MM-yyyy HH:mm:ss</parameter>
					<parameter name="transport.vfs.FailedRecordNextRetryDuration">3000</parameter>
					<parameter name="transport.vfs.ActionAfterProcess">MOVE</parameter>
					<parameter name="transport.vfs.FileURI">file:///tmp/WSO2/Ch19/CSV/In</parameter>
					<parameter name="transport.vfs.MoveAfterFailure">/tmp/WSO2/Ch19/CSV/Original</parameter>
					<parameter name="transport.vfs.ReplyFileName">response.xml</parameter>
					<parameter name="transport.vfs.MoveTimestampFormat">yyy.MMMMM.dd.hh.mm.ss aaa</parameter>
					<parameter name="transport.vfs.DistributedLock">false</parameter>
					<parameter name="transport.vfs.FileNamePattern">.*\.csv</parameter>
					<parameter name="transport.vfs.MoveAfterProcess">/tmp/WSO2/Ch19/CSV/Out</parameter>
					<parameter name="transport.vfs.Locking">disable</parameter>
					<parameter name="transport.vfs.SFTPUserDirIsRoot">false</parameter>
					<parameter name="transport.vfs.FileSortAttribute">none</parameter>
					<parameter name="transport.vfs.FileSortAscending">true</parameter>
					<parameter name="transport.vfs.CreateFolder">true</parameter>
					<parameter name="transport.vfs.Streaming">false</parameter>
					<parameter name="transport.vfs.Build">false</parameter>
					<parameter name="transport.vfs.UpdateLastModified">true</parameter>
				</parameters>
			```


		b. `XMLDataFileProcessIEP`

		* Inbound Endpoint Creation Type: `File`
		* Sequence: `XMLDataFileProcessSeq`
		* Error Sequence: `ErrorProcessSeq` 
		
		Click Finish
		
		* Change InboundEP Mediator Properties:		
			```xml
				<parameters>
					<parameter name="interval">1000</parameter>
					<parameter name="sequential">true</parameter>
					<parameter name="coordination">true</parameter>
					<parameter name="transport.vfs.ContentType">text/xml</parameter>
					<parameter name="transport.vfs.LockReleaseSameNode">false</parameter>
					<parameter name="transport.vfs.AutoLockRelease">false</parameter>
					<parameter name="transport.vfs.ActionAfterFailure">MOVE</parameter>
					<parameter name="transport.vfs.FailedRecordsFileName">vfs-move-failed-records.properties</parameter>
					<parameter name="transport.vfs.FailedRecordsFileDestination">repository/conf/</parameter>
					<parameter name="transport.vfs.MoveFailedRecordTimestampFormat">dd-MM-yyyy HH:mm:ss</parameter>
					<parameter name="transport.vfs.FailedRecordNextRetryDuration">3000</parameter>
					<parameter name="transport.vfs.ActionAfterProcess">MOVE</parameter>
					<parameter name="transport.vfs.FileURI">file:///tmp/WSO2/Ch19/XML/In</parameter>
					<parameter name="transport.vfs.MoveAfterFailure">/tmp/WSO2/Ch19/XML/Original</parameter>
					<parameter name="transport.vfs.ReplyFileName">response.xml</parameter>
					<parameter name="transport.vfs.MoveTimestampFormat">yyy.MMMMM.dd.hh.mm.ss aaa</parameter>
					<parameter name="transport.vfs.DistributedLock">false</parameter>
					<parameter name="transport.vfs.FileNamePattern">.*\.xml</parameter>
					<parameter name="transport.vfs.MoveAfterProcess">/tmp/WSO2/Ch19/XML/Out</parameter>
					<parameter name="transport.vfs.Locking">disable</parameter>
					<parameter name="transport.vfs.SFTPUserDirIsRoot">false</parameter>
					<parameter name="transport.vfs.FileSortAttribute">none</parameter>
					<parameter name="transport.vfs.FileSortAscending">true</parameter>
					<parameter name="transport.vfs.CreateFolder">true</parameter>
					<parameter name="transport.vfs.Streaming">false</parameter>
					<parameter name="transport.vfs.Build">false</parameter>
					<parameter name="transport.vfs.UpdateLastModified">true</parameter>
				</parameters>
			```

		c. `JSONDataFileProcessIEP`

		* Inbound Endpoint Creation Type: `File`
		* Sequence: `JSONDataFileProcessSeq`
		* Error Sequence: `ErrorProcessSeq` 
		
		Click Finish
		
		* Change InboundEP Mediator Properties:		
			```xml
				<parameters>
					<parameter name="interval">1000</parameter>
					<parameter name="sequential">true</parameter>
					<parameter name="coordination">true</parameter>
					<parameter name="transport.vfs.ContentType">application/json</parameter>
					<parameter name="transport.vfs.LockReleaseSameNode">false</parameter>
					<parameter name="transport.vfs.AutoLockRelease">false</parameter>
					<parameter name="transport.vfs.ActionAfterFailure">MOVE</parameter>
					<parameter name="transport.vfs.FailedRecordsFileName">vfs-move-failed-records.properties</parameter>
					<parameter name="transport.vfs.FailedRecordsFileDestination">repository/conf/</parameter>
					<parameter name="transport.vfs.MoveFailedRecordTimestampFormat">dd-MM-yyyy HH:mm:ss</parameter>
					<parameter name="transport.vfs.FailedRecordNextRetryDuration">3000</parameter>
					<parameter name="transport.vfs.ActionAfterProcess">MOVE</parameter>
					<parameter name="transport.vfs.FileURI">file:///tmp/WSO2/Ch19/JSON/In</parameter>
					<parameter name="transport.vfs.MoveAfterFailure">/tmp/WSO2/Ch19/JSON/Original</parameter>
					<parameter name="transport.vfs.ReplyFileName">response.xml</parameter>
					<parameter name="transport.vfs.MoveTimestampFormat">yyy.MMMMM.dd.hh.mm.ss aaa</parameter>
					<parameter name="transport.vfs.DistributedLock">false</parameter>
					<parameter name="transport.vfs.FileNamePattern">.*\.json</parameter>
					<parameter name="transport.vfs.MoveAfterProcess">/tmp/WSO2/Ch19/JSON/Out</parameter>
					<parameter name="transport.vfs.Locking">disable</parameter>
					<parameter name="transport.vfs.SFTPUserDirIsRoot">false</parameter>
					<parameter name="transport.vfs.FileSortAttribute">none</parameter>
					<parameter name="transport.vfs.FileSortAscending">true</parameter>
					<parameter name="transport.vfs.CreateFolder">true</parameter>
					<parameter name="transport.vfs.Streaming">false</parameter>
					<parameter name="transport.vfs.Build">false</parameter>
					<parameter name="transport.vfs.UpdateLastModified">true</parameter>
				</parameters>
			```

	4. Package the artifacts that are created in this project by going to Exporter `Export Artificats Project and Run`. Deselect All previous artifacts and select the ones that we created above by Expanding Inbound EndpointConfigs:
	
		* com.example.sequence_._ErrorProcessSeq
		* com.example.sequence_._XMLDataFileProcessSeq
		* com.example.inbound-endpoint_._CSVDataFileProcessIEP
		* com.example.inbound-endpoint_._XMLDataFileProcessIEP
		* com.example.sequence_._JSONDataFileProcessSeq
		* com.example.sequence_._CSVDataFileProcessSeq
		* com.example.inbound-endpoint_._JSONDataFileProcessIEP

		Click Finish

	The application is successfully deployed when you see the listeners are ready in the console

	```log
		[2022-02-22 23:21:12,901]  INFO {AbstractQuartzTaskManager} - Task scheduled: [ESB_TASK][CSVDataFileProcessIEP-FILE--SYNAPSE_INBOUND_ENDPOINT].
		[2022-02-22 23:21:12,916]  INFO {AbstractQuartzTaskManager} - Task scheduled: [ESB_TASK][XMLDataFileProcessIEP-FILE--SYNAPSE_INBOUND_ENDPOINT].
		[2022-02-22 23:21:12,977]  INFO {AbstractQuartzTaskManager} - Task scheduled: [ESB_TASK][JSONDataFileProcessIEP-FILE--SYNAPSE_INBOUND_ENDPOINT].

	```

	5. Do the following tests for each file type. See how the file copied to the `In` folder got moved to `Out` within 1 second (polling time):
		
		Use Windows Explorer/Finder/Files app to the following folder `/tmp/WSO2/Ch19/`

		a. Go to CSV folder and copy `sample.csv` to `In` folder. The file will be moved to `Out` folder
		```log
			[2022-02-22 23:23:19,219]  INFO {LogMediator} - {inboundendpoint:CSVDataFileProcessIEP} To: , MessageID: urn:uuid:DD330314E1061987CE1645590199208, Direction: request, Envelope: <?xml version='1.0' encoding='utf-8'?><soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"><soapenv:Body><text xmlns="http://ws.apache.org/commons/ns/payload">1,Nelson,Dias,Portugal,Developer&#xd;
			2,Romeo,Santos,RepDominicana,Singer</text></soapenv:Body></soapenv:Envelope>
		```	
		 
		b. Go to XML folder and copy `test.xml` to `In` folder. The file will be moved to `Out` folder
		```log
		[2022-02-22 23:31:28,932]  INFO {LogMediator} - {inboundendpoint:XMLDataFileProcessIEP} To: , MessageID: urn:uuid:DD330314E1061987CE1645590688920, Direction: request, Envelope: <?xml version='1.0' encoding='utf-8'?><soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:wsa="http://www.w3.org/2005/08/addressing"><soapenv:Body>
				<orders>
					<order>
						<country>Portugal</country>
					</order>
				</orders>
			</soapenv:Body></soapenv:Envelope>
		[2022-02-22 23:31:28,960]  INFO {LogMediator} - {inboundendpoint:XMLDataFileProcessIEP} LOG COUNTRY = Portugal
			
		```
		
		c. Go to JSON folder and copy `sample.json` to `In` folder. The file will be moved to `Out` folder
		```log
			[2022-02-22 23:35:49,978]  INFO {LogMediator} - {inboundendpoint:JSONDataFileProcessIEP} To: , MessageID: urn:uuid:DD330314E1061987CE1645590949979, Direction: request, Payload: {  "id": 12345, "id_str": "12345", "array": [ 1, 2, [ [], [{"inner_id": 6789}] ] ], "name": null, "object": {}, "$schema_location": "unknown", "12X12": "image12x12.png" }
			[2022-02-22 23:35:49,992]  INFO {LogMediator} - {inboundendpoint:JSONDataFileProcessIEP} LOG ID = 12345
			
		```

20. Scheduled Tasks (Project folder: ScheduledTask)

	Create task that runs periodically. It can inject messages, define endpoints, proxy services, and so on. This project uses scheduled task to pass the country and location in xml message to a proxy. From proxy, it passed on to the Endpoint then, from Endpoint, it passed to the WeatherAPI. 

	1. Create new integration studio project called `ScheduledTask`

	2. Create new REST API artifacts called `WeatherAPI` with context `API`
		
		a. On the Resource, set the following
		```xml
			<resource methods="GET" protocol="http" uri-template="/weather/{country}/{city}">
		```
		
		b. Add the following mediators to the inSequence
		```xml
				<inSequence>
					<log description="LOG START" level="custom">
						<property name="LOG MESSAGE" value="LOG START API"/>
					</log>
					<payloadFactory description="SET PAYLOAD" media-type="json">
						<format>{
							"country":$1,
							"city": $2,
							"weather": "sunny"
						}</format>
						<args>
							<arg evaluator="xml" expression="$ctx:uri.var.country"/>
							<arg evaluator="xml" expression="$ctx:uri.var.city"/>
						</args>
					</payloadFactory>
					<log description="LOG END" level="custom">
						<property name="LOG MESSAGE" value="LOG END API"/>
					</log>
					<loopback/>
				</inSequence>
		```
		c. Add the following mediator to the outSequence
		```xml
			<outSequence>
            	<respond description="SEND RESPONSE"/>
        	</outSequence>
		```
	3. Create an Endpoint artifact called `WeatherApiEP` 
	
		a. Endpoint Type: `HTTP Endpoint` (This always bite back and easily forgotten to change)

		b. URI Template: `http://localhost:8290/api/weather/{uri.var.countryProxy}/{uri.var.cityProxy}`

	4. Create a Proxy Service artifact called `CustomWeatherApiProxy`

		a. Add the following mediators to the outSequence
		```xml
			<inSequence>
				<property description="SET COUNTRY PROXY" expression="//request/location/country" name="uri.var.countryProxy" scope="default" type="STRING"/>
				<property description="SET CITY PROXY" expression="//request/location/city" name="uri.var.cityProxy" scope="default" type="STRING"/>
				<log description="LOG LOCATION" level="custom">
					<property expression="$ctx:uri.var.countryProxy" name="LOG COUNTRY"/>
					<property expression="$ctx:uri.var.cityProxy" name="LOG CITY"/>
				</log>
				<send>
					<endpoint key="WeatherApiEP"/>
				</send>
			</inSequence>
		```

		b. Add the following mediator to the outSequence
		```xml
			<outSequence>
				<log description="LOG OUTPUT" level="full"/>
			</outSequence>
		```
	
	5. Create Scheduled Task artifact called `InjectRequestWeatherApiTask`. We keep it `Simple`, not using cron method and set the count to `2` and Interval to `10` seconds. This will run the proxy twice within 10 seconds

		a. Once artifact is created, click on `Task Implementation Properties`, set
			
		1. **injectTo** with value `Proxy`

		2. **message** to `XML` with value (if there is any problem, try linearize the xml into single line)
		```xml
			<request>
				<location>
					<city>Lisbon</city>
					<country>Portugal</country>
				</location>
			</request>
		```

		3. **proxyName** with value `CustomWeatherApiProxy`

	6. Export project artifacts and Run
	```log
	[2022-02-27 00:46:36,721]  INFO {AbstractQuartzTaskManager} - Task scheduled: [ESB_TASK][InjectRequestWeatherApiTask].
	[2022-02-27 00:46:37,279]  INFO {LogMediator} - {proxy:CustomWeatherApiProxy} LOG COUNTRY = Portugal, LOG CITY = Lisbon
	[2022-02-27 00:46:37,467]  INFO {TimeoutHandler} - This engine will expire all callbacks after GLOBAL_TIMEOUT: 120 seconds, irrespective of the timeout action, after the specified or optional timeout
	[2022-02-27 00:46:37,674]  INFO {LogMediator} - {api:WeatherAPI} LOG MESSAGE = LOG START API
	[2022-02-27 00:46:37,745]  INFO {LogMediator} - {api:WeatherAPI} LOG MESSAGE = LOG END API
	[2022-02-27 00:46:37,895]  INFO {LogMediator} - {proxy:CustomWeatherApiProxy} To: http://www.w3.org/2005/08/addressing/anonymous, WSAction: , SOAPAction: , MessageID: urn:uuid:cabb9503-19ea-4610-94e0-488512a2473a, correlation_id: c6ff23f0-9363-4c78-a0da-4a3acb1c472c, Direction: response, Payload: {
		"country":Portugal,
		"city": Lisbon,
		"weather": "sunny"
	}
	[2022-02-27 00:46:39,519]  INFO {AuthenticationHandlerAdapter} - User admin logged in successfully
	[2022-02-27 00:46:39,605]  INFO {ServiceComponent} - Initializing Security parameters
	[2022-02-27 00:46:46,732]  INFO {LogMediator} - {proxy:CustomWeatherApiProxy} LOG COUNTRY = Portugal, LOG CITY = Lisbon
	[2022-02-27 00:46:46,743]  INFO {LogMediator} - {api:WeatherAPI} LOG MESSAGE = LOG START API
	[2022-02-27 00:46:46,744]  INFO {LogMediator} - {api:WeatherAPI} LOG MESSAGE = LOG END API
	[2022-02-27 00:46:46,767]  INFO {LogMediator} - {proxy:CustomWeatherApiProxy} To: http://www.w3.org/2005/08/addressing/anonymous, WSAction: , SOAPAction: , MessageID: urn:uuid:0ca99bd0-f233-4cfd-b9a4-21a999b4e3ae, correlation_id: 11d316fe-31c4-4687-b361-0547b434468a, Direction: response, Payload: {
		"country":Portugal,
		"city": Lisbon,
		"weather": "sunny"
	}
	```


## Section 3: Message processing units

22. Send Mediator (Project folder: HealthcareProject)

	This project is allowing healthcare patients to search for specialists by providing the category and return the list back to the patient. One thing to note that send mediator's role is to send the request message to the healthcare service endpoint. It used to send messages to synapse engine. It also copies any message's contexts and properties from the current message context to the replied message received on the execution of Send operation. Since we're not defining the receiving sequence, the default behavior will use outSequence to handle the reponse.

	1. Create New Integration Project called `HealthcareProject` with the following modules: ESB Configs, Composite Exporter, Registry Resources, Connector Exporter.

	2. Start the web services provided for this chapter. We can use java decompiler app to view the `Hospital-Service-JDK11-2.0.0.jar` code that run these backend services. The instructor uses [JD-GUI app](https://github.com/java-decompiler/jd-gui).
		```shell		
		# Go to the resources folder
		java8 -jar Resources/Ch_22_Send_Mediator/Hospital-Service-JDK11-2.0.0.jar
		# To test out, run this in another terminal
		curl -v GET "http://localhost:9090/healthcare/surgery"
		curl -v GET "http://localhost:9090/healthcare/ent"
		```	

	3. Back to WSO2, create Endpoints artifacts that point to the java services

		a. Endpoint Type: `HTTP Endpoint` 
		
		b. URI Template: `http://localhost:9090/healthcare/{uri.var.category}`

	4. Create REST API artifacts called `HealthcareAPI` with context `/healthcare`. The API handles client request. Add the following mediators to the API. 
		```xml
		<resource methods="GET" uri-template="/querydoctor/{category}">
			<inSequence>
				<log description="LOG WELCOME" level="custom">
					<property name="LOG MESSAGE" value="WELCOME TO HEALTHCARE SERVICE"/>
				</log>
				<log description="LOG BEFORE" level="custom">
					<property name="LOG MESSAGE" value="LOG BEFORE SEND"/>
				</log>
				<send>
					<endpoint key="QueryDoctorEP"/>
				</send>
			</inSequence>
			<outSequence>
				<log description="LOG END" level="custom">
					<property name="LOG MESSAGE" value="LOG END"/>
				</log>
				<send/>
			</outSequence>
			<faultSequence/>
		</resource>
		```
	5. Export Project Artifacts and Run
		```shell
			curl -v GET "http://localhost:8290/healthcare/querydoctor/surgery"

			#WSO2 Logs
			2022-02-28 00:58:29,035]  INFO {LogMediator} - {api:HealthcareAPI} LOG MESSAGE = WELCOME TO HEALTHCARE SERVICE
			[2022-02-28 00:58:29,036]  INFO {LogMediator} - {api:HealthcareAPI} LOG MESSAGE = LOG BEFORE SEND
			[2022-02-28 00:58:29,052]  INFO {TimeoutHandler} - This engine will expire all callbacks after GLOBAL_TIMEOUT: 120 seconds, irrespective of the timeout action, after the specified or optional timeout
			[2022-02-28 00:58:29,108]  INFO {LogMediator} - {api:HealthcareAPI} LOG MESSAGE = LOG END
		```	





## Section 4: Message exit points


## Section 5: Data Services


