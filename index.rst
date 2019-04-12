.. title:: Xi IoT - Data Pipelines - Getting Started Guide

.. toctree::
  :maxdepth: 2
  :caption:     Contents
  :name: _req-labs
  :hidden:

  .. example/index
  contents/lab

.. _welcome:

**Welcome**
===========
Welcome to the Xi IoT - Data Pipelines - Getting Started Guide, v0.2.


**Introducing Data Pipelines**
===============================

The main steps in this guide are excerpts from the `Xi IoT Infrastructure Admin Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Xi-IoT-Infra-Admin-Guide:Xi-IoT-Infra-Admin-Guide>`__, available from the Nutanix Support Portal.

Data Pipelines are paths for data that include:

-  **Input**. An existing data source or real-time data stream.

-  **Transformation**. Code block such as a script defined in a Function to process or transform input data.

-  **Output**. A destination for your data. Publish data to the cloud or cloud data service (such as AWS Simple  Queue Service) at the edge.

They also enable you to process and transform captured data for further consumption or processing.

Data pipelines have the following components used in the examples in this guide:

-  Data Sources (defined as MQTT in this guide)

-  Runtime Environments

-  Functions

**Using MQTT Data Sources in Data Pipelines**
---------------------------------------------

**What is MQTT?**
~~~~~~~~~~~~~~~~~

If you are looking to understand the internals of how MQTT works, please read the 10 part series on `MQTT Essentials <https://www.hivemq.com/tags/mqtt-essentials/>`__ by HiveMQ.

**Adding a Data Source**
------------------------

You can add one or more data sources (a collection of sensors, gateways, or other input devices providing data) to associate with an edge.

Each defined data source consists of:

-  Data source type (sensor, input device like a camera, or gateway) - the origin of the data

-  Communication protocol typically associated with the data source

-  Authentication type to secure access to the data source and data

-  One or more fields specifying the data extraction method - the data pipeline specification

-  Categories which are attributes that can be metadata you define to associate with the captured data

**Add a Data Source - MQTT**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add one or more data sources using the Message Queuing Telemetry Transport lightweight messaging protocol (MQTT) to associate with an
edge. Any device supporting MQTT with the ability authenticate via certificates X.509 certificates is supported.

Certificates downloaded from the Xi IOT management console have an expiration date 30 years from the certificate creation date. Download
the certificate ZIP file each time you create an MQTT data source. Nutanix recommends that you use each unique set of these security certificates and keys for any MQTT data sources you add.

When naming entities, Up to 200 alphanumeric characters are allowed.

1. Click **Infrastructure** > **Data Sources** > **Add Data Source**.

2. Select a data source type: **Sensor** or **Gateway**.

   1. **Name** your data source.

   2. **Associated edge**. Select an edge for your data source.

   3. Select **Protocol** > **MQTT**.

   4. **Authentication type**. When you select the **MQTT** protocol, the **Certificate** authentication type is selected automatically to authenticate the connection between your data source and the edge.

   5. Click **Generate Certificates**, then click **Download** to download a ZIP file that contains X.509 sensor certificate (public key) and its private key and Root CA certificates.

   6. See `Certificates Used with MQTT Data Sources <https://portal.nutanix.com/#/page/docs/details?targetId=Xi-IoT-Infra-Admin-Guide:edg-iot-certificates-c.html>`__.

3. Click **Next** to go to **Data Extraction** to specify the datafields to extract from the data source.

**Data Extraction - MQTT**
--------------------------

1. Click **Add New Field**.

   1. **Name**. Enter a relevant name for the data source field.

   2. **Add MQTT Topic**. Enter a topic (a case-sensitive UTF-8 string). For example, /temperature/frontroom for a temperature sensor located in a specific room.

   3. Click the check icon to add the data source field.

2. Click **Add New Field** to add another topic or click **Next** to go to the **Category Assignment** panel.

**Categories - MQTT**
---------------------

Categories enable you to define metadata associated with captured files from the data source.

1. Select **All Fields** from **Select Fields**.

2. Select **Category** > **Data Type**.

3. Select one of the Categories (as defined by `Categories <https://portal.nutanix.com/#/page/docs/details?targetId=Xi-IoT-Infra-Admin-Guide:edg-iot-add-category-t.html>`__ you have created) in the third selection menu.

4. Click **Add**.

**MQTT Client Samples for Testing**
-----------------------------------

If you are looking to understand the internals of how MQTT works, please read the 10 part series on `MQTT Essentials <https://www.hivemq.com/tags/mqtt-essentials/>`__ by HiveMQ.

**Javascript**
~~~~~~~~~~~~~~

Please refer to `mqtt package <https://www.npmjs.com/package/mqtt>`__ and examples
`here <https://github.com/mqttjs/MQTT.js/blob/master/examples/client/secure-client.js>`__ for creating secure mqtt clients in javascript.

**Python 2**
~~~~~~~~~~~~

   Prerequisites

-  A Nutanix edge with an IP address onboarded to Xi IoT

-  X509 certificates generated using Xi IoT

-  Python 2.7.10

-  pip 10.0.1 (python 2.7)

-  paho-mqtt. Install it for python 2.7.10 using the following command: sudo pip2.7 install paho-mqtt

..

   Sample

   Below is a simple example that shows how to connect to an mqtt  broker, publish a single message to a specific topic and receive the published message back.

   .. code-block:: python2

    # Example code to connect, publish and subscribe from a mqtt client
    # For the example to work:
    # 1. create a dir named 'certs' under $PWD and copy the certs
    #    generated using Xi IoT SaaS Portal.
    # 2. Modify the 'broker_address' variable to point to the edge
    #    ip address that is being used for the tests.

    import paho.mqtt.client as mqttClient
    import time
    import ssl

    def on_connect(client, userdata, flags, rc):
        if rc == 0:
            print("Connected to broker")
            global Connected
            Connected = True                #Signal connection 
        else:
            print("Connection failed")

    def on_publish(client, userdata, result):
      print "Published!"

    def on_message(client, userdata, message):
        print "New message received!"
        print "Topic: ", message.topic
        print "Message: ", str(message.payload.decode("utf-8"))

    def main():
        global Connected
        Connected = False
        # IP address of the edge. Modify this.
        broker_address= "<edge_ip>"
        port = 1883
        # NOTE: For data pipelines to receive MQTT messages, topic should
        #       be the same as that specified when creating the MQTT datasource.
        topic = "test"

        client = mqttClient.Client()
        # Set callbacks for connection event, publish event and message receive event
        client.on_connect = on_connect
        client.on_publish = on_publish
        client.on_message = on_message
        client.tls_set(ca_certs="certs/ca.crt", certfile="certs/client.crt", keyfile="certs/client.key", cert_reqs=ssl.CERT_REQUIRED, tls_version=ssl.PROTOCOL_TLSv1_2, ciphers=None)
        # Set this to ignore hostname only. TLS is still valid with this setting.
        client.tls_insecure_set(True)
        client.connect(broker_address, port=port)
        client.subscribe(topic)
        client.loop_start()

        # Wait for connection
        while Connected != True:    
            print "Connecting..."
            time.sleep(1)


        try:
            client.publish(topic, "Hello, World!")
            time.sleep(5)
        except KeyboardInterrupt:
            client.disconnect()
            client.loop_stop()

    if __name__ == "__main__":
        main()


   Running the example

1. Download the certificates from Xi IoT and store them locally under certs. directory. Name the files as follows:

 -  ca.crt - Root CA certificate

 -  client.crt - client certificate

 -  client.key - client private key

2. Modify broker_address to point to the Xi IoT edge IP address.

..

   Run the example as follows:

   .. code-block:: python2
   
    $ python2.7 mqtt-example.py

   Expected output:

   .. code-block:: bash
   
    Connecting...
    Connected to broker
    Published!
    New message received!
    Topic: test
    Message: Hello, World!

**Runtime Environments**
=======================

A runtime environment is a command execution environment to run applications written in a particular language or associated with a specific Docker registry or file. 
Each Function added to a Data Pipeline is executed via its own specified Runtime Environment.

Xi IoT includes standard runtime environments including but not limited to the following. These runtimes are read-only and cannot be edited, updated, or deleted by users. They are available to all projects, functions, and associated container registries.

-  **Golang**

-  **NodeJS**

   -  Node.js functions can be run in context of a data pipeline. A transformation function must accept context as well as message payload as parameters. Context can be used to query function parameters passed in when function has been instantiated.
      Moreover context is used to send messages to next stage in data pipeline.

   -  Following is a basic Node.js function template:
   
      .. code-block:: NodeJS

       function main(ctx, msg) {
           return new Promise(function(resolve, reject) {
              // log list of transformation parameters
              console.log("Config", ctx.config)
              // log length of message payload
              console.log(msg.length)
              // forward message to next stage in pipeline
              ctx.send(msg)
             // complete promise
              resolve()
           })
        }

       exports.main = main

   All functions must export main which returns a promise.

   Expected console output:

   .. code-block:: bash
   
    Config { IntParam: '42', StringParam: 'hello' }
    2764855

   .. note::

      Packages available in NodeJS Runtime

      -  alpine-baselayout
      -  alpine-keys
      -  apk-tools
      -  busybox
      -  libc-utils
      -  libgcc
      -  libressl2.5-libcrypto
      -  libressl2.5-libssl
      -  libressl2.5-libtls
      -  libstdc++
      -  musl
      -  musl-utils
      -  scanelf
      -  ssl_client
      -  zlib

-  **Python 2**

   -  Functions can be executed in data pipelines to transform and filter data. Transformations are functions used to process single messages and optionally forward them to next stage in data pipeline. The next stage could be another transformation or destination of the data pipeline on edge or in the cloud.
      Transformation can accept parameters. In Python parameters are passed as dictionary to transformation. The following script demonstrates some basic concepts:

..

   import logging

   # Python function are invoked with context and message payload.

   # The context can be used to retrieve metadata about the message and
   allows

   # function to send mesagges to next stage in stream. In this sample
   we just

   # log message payload and forward it as is to next stage.

   def main(ctx, msg):

   logging.info("Parameters: %s", ctx.get_config())

   logging.info("Process %d bytes from %s at %s", msg, ctx.get_topic(),
   ctx.get_timestamp())

   # Forward to next stage in pipeline.

   ctx.send(msg)

   Pass two parameters to the function:

-  MyStringParam like the name suggests is a parameter of type string.

-  MyIntParam is a number.

..

   The function would produce the following console output when
   processing images from a camera:

   [2019-03-12 04:57:26,820 root INFO] Parameters: {u'MyIntParam':
   u'42', u'MyStringParam': u'hello'}

   [2019-03-12 04:57:26,820 root INFO] Process 2764855 bytes from
   rtsp://184.72.239.149:554/vod/mp4:BigBuckBunny_175k.mov at
   1552366646754939017

   **Methods provided by ctx**

-  **ctx.get_config()** - returns a dict of parameters passed to the function.

-  **ctx.get_topic()** - returns the topic (string) on which the current message was received. In this case, it is the topic is set to RTSP topic from which image has been received.

-  **ctx.get_timestamp()** - returns the time in nanoseconds since epoch (Jan 1st, 1970 UTC).

-  **ctx.send()** - Takes bytes as input and forwards it to the next stage in the pipeline. If the input is not of type bytes, an error is thrown and a corresponding alert is raised in Xi IoT.

..

   **In memory caching**

   Unlike other serverless frameworks, state can be cached in running
   transformation to allow for aggregation across multiple messages.
   Following is a demonstration of state in functions:

   import logging

   counter=0

   def main(ctx, msg):

   global counter

   logging.info("This is message number %d", counter)

   counter+=1

   # Forward to next stage in pipeline.

   ctx.send(msg)

   Script produces following output:

   [2019-03-12 05:19:04,844 root INFO] This is message number 0

   [2019-03-12 05:19:05,846 root INFO] This is message number 1

   [2019-03-12 05:19:06,836 root INFO] This is message number 2

   [2019-03-12 05:19:07,837 root INFO] This is message number 3

   [2019-03-12 05:19:08,838 root INFO] This is message number 4

   The data pipeline has been configured to sample every second.

   Transformation are not limited to just filter or pass thru messages.
   A transformation can send as many messages to next stage in pipeline
   as required by using context:

   import logging

   # Transformation can send more messages than they receive.

   def main(ctx, msg):

   logging.info("Process %d bytes from %s at %s", len(msg),
   ctx.get_topic(), ctx.get_timestamp())

   m = len(msg) / 2

   # split message in two halves

   ctx.send(msg[:m])

   ctx.send(msg[m:])

   Logs will reflect how message have been split:

   [19-03-12 05:30:51,696 root INFO] Process 2764855 bytes from
   rtsp://184.72.239.149:554/vod/mp4:BigBuckBunny_175k.mov

   [2019-03-12 05:30:51,697 root INFO] Send 1382427 bytes

   [2019-03-12 05:30:51,697 root INFO] Send 1382428 bytes

-  Packages available in Python 2 Runtime

-  backports-abc 0.5

-  elasticsearch 6.3.1

-  elasticsearch-dsl 6.3.1

-  futures 3.2.0

-  ipaddress 1.0.22

-  kafka-python 1.4.4

-  msgpack 0.5.6

-  nats-client 0.8.2

-  paho-mqtt 1.4.0

-  pip 18.1

-  prometheus-client 0.5.0

-  protobuf 3.6.1

-  python-dateutil 2.7.5

-  setuptools 40.6.3

-  singledispatch 3.4.0.3

-  six 1.12.0

-  tornado 5.1.1

-  urllib3 1.24.1

-  virtualenv 16.2.0

-  wheel 0.32.3

-  requests 2.20.1

-  **Python 3**

   -  Packages available in Python 3 Runtime

-  asyncio-nats-client 0.8.2

-  elasticsearch 6.3.1

-  elasticsearch-dsl 6.3.1

-  kafka-python 1.4.4

-  msgpack 0.5.6

-  paho-mqtt 1.4.0

-  pip 18.1

-  prometheus-client 0.5.0

-  protobuf 3.6.1

-  python-dateutil 2.7.5

-  setuptools 40.6.3

-  six 1.12.0

-  urllib3 1.24.1

-  wheel 0.32.3

-  requests 2.20.1

-  **Tensorflow Python (Python 2)**

   -  Packages available in Tensorflow Python runtime

      -  Absl-py 0.1.10

      -  astor 0.6.2

      -  astroid 1.6.1

      -  backports-abc 0.5

      -  backports.functools-lru-cache 1.5

      -  backports.shutil-get-terminal-size 1.0.0

      -  backports.weakref 1.0.post1

      -  bleach 1.5.0

      -  configparser 3.5.0

      -  cycler 0.10.0

      -  decorator 4.2.1

      -  elasticsearch 6.3.1

      -  elasticsearch-dsl 6.3.1

      -  entrypoints 0.2.3

      -  enum34 1.1.6

      -  funcsigs 1.0.2

      -  functools32 3.2.3.post2

      -  futures 3.2.0

      -  gast 0.2.0

      -  grpcio 1.10.0

      -  h5py 2.7.1

      -  html5lib 0.9999999

      -  imutils 0.4.5

      -  ipaddress 1.0.22

      -  ipykernel 4.8.2

      -  ipython 5.5.0

      -  ipython-genutils 0.2.0

      -  ipywidgets 7.1.2

      -  isort 4.3.3

      -  Jinja2 2.10

      -  jsonschema 2.6.0

      -  jupyter 1.0.0

      -  jupyter-client 5.2.3

      -  jupyter-console 5.2.0

      -  jupyter-core 4.4.0

      -  kafka-python 1.4.4

      -  kiwisolver 1.0.1

      -  lazy-object-proxy 1.3.1

      -  Markdown 2.6.11

      -  MarkupSafe 1.0

      -  matplotlib 2.2.2

      -  mccabe 0.6.1

      -  mistune 0.8.3

      -  mock 2.0.0

      -  msgpack 0.5.6

      -  nats-client 0.8.2

      -  nbconvert 5.3.1

      -  nbformat 4.4.0

      -  notebook 5.4.1

      -  numpy 1.14.0

      -  opencv-python 3.4.0.12

      -  paho-mqtt 1.4.0

      -  pandas 0.22.0

      -  pandocfilters 1.4.2

      -  pathlib2 2.3.0

      -  pbr 4.0.0

      -  pexpect 4.4.0

      -  pickleshare 0.7.4

      -  Pillow 5.0.0

      -  pip 18.1

      -  prometheus-client 0.5.0

      -  prompt-toolkit 1.0.15

      -  protobuf 3.5.1

      -  ptyprocess 0.5.2

      -  Pygments 2.2.0

      -  pylint 1.8.2

      -  pyparsing 2.2.0

      -  python-dateutil 2.7.5

      -  pytz 2018.3

      -  pyzmq 17.0.0

      -  qtconsole 4.3.1

      -  scandir 1.7

      -  scikit-learn 0.19.1

      -  scipy 1.0.0

      -  Send2Trash 1.5.0

      -  setuptools 40.6.3

      -  simplegeneric 0.8.1

      -  singledispatch 3.4.0.3

      -  six 1.11.0

      -  sklearn 0.0

      -  subprocess32 3.2.7

      -  tensorboard 1.7.0

      -  tensorflow 1.7.0

      -  termcolor 1.1.0

      -  terminado 0.8.1

      -  testpath 0.3.1

      -  tornado 5.1.1

      -  traitlets 4.3.2

      -  urllib3 1.24.1

      -  virtualenv 16.2.0

      -  wcwidth 0.1.7

      -  webencodings 0.5.1

      -  Werkzeug 0.14.1

      -  wheel 0.32.3

      -  widgetsnbextension 3.1.4

      -  wrapt 1.10.11

You can add your own custom runtime environment for use by all or specific projects, functions, and container registries.

**Note:** Custom Golang runtime environments are not supported. Use the provided standard Golang runtime environment in this case.

**Building a Custom Runtime Environment**
-----------------------------------------

You may need a custom runtime for some third party packages or OS distributions (like Linux) which might have dependencies not covered with the built-in Xi IoT runtimes.

Like the built-in runtime environments, custom runtimes are docker images that can run functions. **A runtime container image must include the Xi IoT language-specific runtime bundle**.

The bundle's runtime environment is responsible for:

-  Bootstraping the container by downloading the script assigned to that container at runtime

-  Receiving messages and events

-  Providing the API necessary to inspect and forward messages

-  Reporting statistics and alerts to Xi IoT control plane

Nutanix provides custom runtime support for three languages:

-  Python 2
      (https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python2-env.tgz)

-  Python 3
      (https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz)

-  NodeJS
      (https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/node-env.tgz)

Python 2 is distinguished from Python 3, as Python 3 syntax and libraries are not backward-compatible.

This sample Dockerfile builds a custom runtime environment able to run Python 3 functions:

   FROM python:3.6

   RUN python -V

   # Check Python version

   RUN python -c 'import sys; sys.exit(sys.version_info.major != 3)'

   # We need Python runtime environment to execute Python functions.

   RUN wget
   https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz

   RUN tar xf /python-env.tgz

   # Bundle does not come with all required packages but defines them as PIP dependencies

   RUN pip install -r /python-env/requirements.txt

   # In this example we install Kafka client for Python as additional 3rd party software

   RUN pip install kafka-python

   # Containers should NOT run as root as a good practice

   # We mandate all runtime containers to run as user 10001

   USER 10001

   # Finally run Python function worker which pull and executes functions.

   CMD ["/python-env/run.sh"]

Build this container as usual by invoking "docker build".

   $ docker build -t edgecomputing/sample-env -f Dockerfile .

   Sending build context to Docker daemon 2.56kB

   Step 1/9 : FROM python:3.6

   Step 2/9 : RUN python -V

   Step 3/9 : RUN python -c 'import sys; sys.exit(sys.version_info.major
   != 3)'

   Step 4/9 : RUN wget
   https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz

   Step 5/9 : RUN tar xf /python-env.tgz

   Step 6/9 : RUN pip install -r /python-env/requirements.txt

   Step 7/9 : RUN pip install kafka-python

   Step 8/9 : USER 10001

   Step 9/9 : CMD ["/python-env/run.sh"]

   Removing intermediate container 52d45f3db900

   ---> 95a878cde355

   Successfully built 95a878cde355

   Successfully tagged edgecomputing/sample-env:latest

Upload the docker image to a container registry. AWS Elastic Containeregistry (ECR) is used in this example:

   $ docker tag edgecomputing/sample-env:latest
   $DOCKER_REPO/sample-env:latest

   $ docker push $DOCKER_REPO/sample-env:latest

**Example Custom Runtimes**

   **Node.js**

   FROM node:9

   RUN wget
   https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/node-env.tgz

   RUN tar xf /node-env.tgz

   WORKDIR /node-env

   RUN npm install

   # Containers should NOT run as root as a good practice

   USER 10001

   CMD ["/node-env/run.sh"]

   **Python 3**

   FROM python:3.6

   RUN python -V

   # Check Python version

   RUN python -c 'import sys; sys.exit(sys.version_info.major != 3)'

   # We need Python runtime environment to execute Python functions.

   RUN wget
   https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz

   RUN tar xf /python-env.tgz

   # Bundle does not come with all required packages but defines them as PIP dependencies

   RUN pip install -r /python-env/requirements.txt

   # In this example we install Kafka client for Python as additional 3rd party software

   RUN pip install kafka-python

   # Containers should NOT run as root as a good practice

   # We mandate all runtime containers to run as user 10001

   USER 10001

   # Finally run Python function worker which pull and executes functions.

   CMD ["/python-env/run.sh"]

**Creating a Runtime Environment**
----------------------------------

This topic describes how to create a runtime environment and assumes you are already logged on to the Xi IoT management console.

-  Click **X** or **Cancel** to exit this task.

1. Click the menu button, then click **Apps and Data** > **Runtime Environments** > **Create**.

2. **Name**. Name the runtime environment. Up to 75 alphanumeric characters are allowed.

3. **Description**. Provide a relevant description.

4. Select a specific project to make your runtime environment available to that project.

5. **Container Registry**. Select an existing container registry profile.

6. **Container Image Path**. Provide a container image URL. For example, https://aws_account_id.dkr.ecr.region.amazonaws.com/image:imagetag

7. **Language**. Select the scripting language for the runtime: golang, python, node.

8. [Optional] Click **Add Docker Profile** to choose a Docker file for the container image, then click **Done**.

9. Click **Create**.

**Editing a Runtime Environment**
---------------------------------

This topic describes how to edit a runtime environment and assumes you are already logged on to the Xi IoT management console.

-  You cannot edit standard runtime environments included with Xi IoT.

-  Click **X** or **Cancel** to exit this task.

1.  Click the menu button, then click **Apps and Data** > **Runtime Environments**.

2.  Click a runtime environment in the list.

3.  The dashboard is displayed along with an **Edit** button.

4.  Click **Edit**.

5.  **Name**. Name the runtime environment. Up to 75 alphanumeric characters are allowed.

6.  **Description**. Provide a relevant description.

7.  Select a project to make your runtime environment available to that project.

8.  **Container Registry**. Select an existing container registry profile.

9.  **Container Image Path**. Provide a container image URL. For example,
       https://aws_account_id.dkr.ecr.region.amazonaws.com/image:imagetag

10. **Language**. Select the scripting language for the runtime: golang, python, node.

11. [Optional] **Remove** or **Edit** any existing Docker file.

12. After editing a file, click **Done**.

13. Click **Add Docker Profile** to choose a Docker file for the container image.

14. Click **Update**.

**Removing a Runtime Environment**
----------------------------------

This topic describes how to delete a runtime environment and assumes you have already logged on to the Xi IoT management console.

1. Click the menu button, then click **Apps and Data** > **Runtime Environments**.

2. Select a runtime environment, click **Remove**, then click **Remove** again to confirm.

3. The **Runtime Environments** page lists any remaining runtime environments.

4. The **Data Pipelines** page lists any remaining data pipelines.

**Functions**
=============

A function is code used to perform one or more tasks. Script languages include Python, Golang, and Node.js. A script can be as simple as text processing code or it could be advanced code implementing artificial intelligence, using popular machine learning frameworks like Tensorflow.

An infrastructure administrator or project user can create a function, and later can edit or clone it. You cannot edit a function that is used by an existing data pipeline. In this case, you can clone it to make an editable copy.

-  When you create, clone, or edit a function, you can define one or more parameters.

-  When you create a data pipeline, you define the values for the parameters when you specify the function in the pipeline.

-  Data pipelines can share functions, but you can specify unique parameter values for the function in each data pipeline.

**Creating a Function**
-----------------------

This topic describes how to create a function and assumes you are already logged on to the Xi IoT management console.

-  Click **X** or **Cancel** to exit this task.

1. Click the menu button, then click **Apps and Data** > **Functions** > **Create**.

2. **Name**. Name the function. Up to 200 alphanumeric characters are allowed.

3. **Description**. Provide a relevant description for your function.

4. Select a project from the **Project** drop-down menu.

5. **Language**. Select the scripting language for the function: golang, python, node.

6. **Runtime Environment**. Select the runtime environment for the function. A default runtime environment is already selected in most cases depending on the **Language** you have selected.

7. Click **Next**.

8. Add function code:

   1. Click **Choose File** to navigate to a file containing your function code.

   2. After uploading the file, you can edit the file contents. You can also choose the contrast levels **Normal**, **Dark**, or **High Contrast Dark**.

   3. Click **Add Parameter** to define and use one or more parameters for data transformation in the function. Click the check icon to add each parameter.

9. Click **Create**.

**Editing a Function**
----------------------

This topic describes how to edit an existing function and assumes you are already logged on to the Xi IoT management console.

-  Click **X** or **Cancel** to exit this task.

-  Other than the name and description, you cannot edit a function that is in use by an existing data pipeline. In this case, you can clone a function to duplicate it. See `Cloning a Function <https://portal.nutanix.com/#/page/docs/details?targetId=Xi-IoT-Infra-Admin-Guide:edg-iot-functions-clone-t.html>`__.

1.  Click the menu button, then click **Apps and Data** > **Functions**.

2.  Click a function in the list.

3.  The function dashboard is displayed along with an **Edit Function** button.

4.  **Name**. Update the function name. Up to 200 alphanumeric characters are allowed.

5.  **Description**. Provide a relevant description for your function.

6.  You cannot change the function's **Project**.

7.  **Language**. Select the scripting language for the function: golang, python, node.

8.  **Runtime Environment**. Select the runtime environment for the function. A default runtime environment is already selected in most cases depending on the **Language** you have selected.

9.  Click **Next**.

10. If you want to choose a different file or edit the existing function code:

    1. Click **Choose File** to navigate to a file containing your function code.

    2. After uploading the file, you can edit the file contents. You can also choose the contrast levels **Normal**, **Dark**, or **High Contrast Dark**.

    3. Click **Add Parameter** to define and use one or more parameters for data transformation in the function. Click the check icon to add each parameter.

11. Click **Update**.

**Cloning a Function**
----------------------

This topic describes how to clone an existing function and assumes you are already logged on to the Xi IoT management console.

Click **X** or **Cancel** to exit this task.

1.  Click the menu button, then click **Apps and Data** > **Functions**.

2.  Click a function in the list.

3.  The function dashboard is displayed along with an **Clone** button.

4.  **Name**. Update the function name. Up to 200 alphanumeric characters are allowed.

5.  **Description**. Provide a relevant description for your function.

6.  You cannot change the function's **Project**.

7.  **Language**. Select the scripting language for the function: golang, python, node.

8.  **Runtime Environment**. Select the runtime environment for the function. A default runtime environment is already selected in most cases depending on the **Language** you have selected.

9.  Click **Next**.

10. If you want to choose a different file or edit the existing function code:

    1. Click **Choose File** to navigate to a file containing your function code.

    2. After uploading the file, you can edit the file contents. You can also choose the contrast levels **Normal**, **Dark**, or **High Contrast Dark**.

    3. Click **Add Parameter** to define and use one or more parameters for data transformation in the function. Click the check icon to add each parameter.

11. Click **Create**.

**Removing a Function**
-----------------------

This topic describes how to delete a function and assumes you have already logged on to the Xi IoT management console.

You cannot remove a function that is associated with a data pipeline or realtime data stream.

1. Click the menu button, then click **Apps and Data** > **Functions**.

2. Select a function, click **Remove**, then click **Remove** again to confirm.

3. The **Functions** page lists any remaining functions.

**Data Pipelines**
==================

**Data Pipeline Visualization**
-------------------------------

After you have created one or more data pipelines, the **Data Pipelines** > **Visualization** page shows data pipelines and the relationship among data pipeline components.

You can view data pipelines associated with a Xi Edge by clicking the filter icon under each title (Data Sources, Data Pipelines on Edge, Data Pipelines on Cloud) and selecting one or more Xi Edges in the drop-down list.

|Shows the existing data pipelines and the relationship among data pipeline components.|

**Figure. Data Pipeline Visualization**

.. |Shows the existing data pipelines and the relationship among data pipeline components.| image:: http://download.nutanix.com/documentation/hosted/images/edge-pipeline-visual.png
   :width: 6.5in
   :height: 4.29167in
   :target: http://download.nutanix.com/documentation/hosted/images/edge-pipeline-visual.png




**Creating a Data Pipeline**
----------------------------

**Before you begin**
--------------------

You must have already created at least one of each: project, data source, function, and category. Also, a cloud profile is required for cloud data destinations or edge endpoints.

-  Click **X** or **Cancel** to exit this task.

1. Click the menu button, then click **Apps and Data** > **Data Pipelines** > **Create**.

2. Select a project, then click **Next**.

3. **Data Pipeline Name**. Name your data pipeline. Up to 63 lowercase alphanumeric characters and the dash character (-) are allowed.

**Input - Add a Data Source**
-----------------------------

Click **Add Data Source**, then select **Data Source**.

1. Select a data source **Category** and one related **Value**.

2. [Optional] Click **Add new** to add another **Category** and **Value**. Continue adding as many as you have defined.

3. Click **X** to delete a data source category and value.

**Transformation - Add a Function**
-----------------------------------

1. Click **Add Function** and select an existing Function.

2. Define a data **Sampling Interval** by selecting **Enable**.

   1. Enter a interval value.

   2. Select an interval: **Millisecond**, **Second**, **Minute**, **Hour**, or **Day**.

3. Enter a value for any parameters defined in the function.

4. If no parameters are defined, this field's name is shown as parens characters. **( )**

5. [Optional] Click the plus sign (**+**) to add another function.

6. This step is useful if you want to chain functions together, feeding transformed data from one function to another, and so on.

7. You can also create a function here by clicking **Upload**, which displays the **Add Function** page.

   3.  **Name** your function. Up to 200 alphanumeric characters are allowed.

   4.  **Description**. Provide a relevant description for the function.

   5.  **Project**. Select the project you first selected for this data pipeline.

   6.  **Language**. Select the scripting language for the function: golang, python, node.

   7.  **Runtime Environment**. Select the runtime environment for the function. A default runtime environment is already selected in most cases depending on the **Language** you have selected.

   8.  Click **Next**.

   9.  Click **Choose File** to navigate to a file containing your function code.

   10. After uploading the file, you can edit the file contents. You can also choose the contrast levels **Normal**, **Dark**, or
       **High Contrast Dark**.

   11. Click **Add Parameter** to define and use any parameters for the data transformation in the function.

   12. Click **Create**.

**Output - Add a Destination**
------------------------------

1. Click **Add Destination** to specify where the transformed data is to be output.

   1. **Destination**. Select **Edge** or **Cloud**.

2. If you select **Destination** > **Edge**:

   2. **Endpoint Type**. Select **MQTT** or **Realtime Data Stream**.

   3. **Note:** Elasticsearch and Kafka are available for tech preview use as edge endpoint types in data pipelines. Do not use tech preview features in production environments.

   4. **Name** your destination. Up to 200 lowercase alphanumeric characters and the dash character (-) are allowed.

3. If you select **Destination** > **Cloud** and Cloud Type as **AWS**, complete the following fields:

   5. **Cloud Profile**. Select your existing profile.

   6. See the topic Create Your Cloud Profile in the `Xi IoT Administration Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Xi-IoT-Infra-Admin-Guide:Xi-IoT-Infra-Admin-Guide>`__.

   7. **Endpoint Type**. Select an existing endpoint type.

   8. **Endpoint Name**. Name your endpoint. Up to 200 alphanumeric characters and the dash character (-) are allowed.

   9. **Region**. Select an AWS region.

4. If you select **Destination** > **Cloud** and Cloud Type as **GCP**, complete the following fields:

   10. **Cloud Profile**. Select your existing profile.

   11. See the topic Create Your Cloud Profile in the `Xi IoT Administration Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Xi-IoT-Infra-Admin-Guide:Xi-IoT-Infra-Admin-Guide>`__.

   12. **Endpoint Type**. Select an existing endpoint type.

   13. **Endpoint Name**. Name your endpoint. Up to 200 alphanumeric characters and the dash character (-) are allowed.

   14. **Region**. Select a GCP region.

5. Click **Create**.

**Editing a Data Pipeline**
---------------------------

This topic describes how to update a data pipeline and assumes you have already logged on to the Xi IoT management console.

Click **X** or **Cancel** to exit this task.

1. Click the menu button, then click **Apps and Data** > **Data Pipelines**.

2. Click a data pipeline in the list.

3. The data pipeline dashboard is displayed along with an **Edit** button.

4. Click **Edit**.

5. You cannot update the data pipeline name.

**Input - Edit a Data Source**
------------------------------

You can do any of the following:

1. Select a different **Data Source** or **Realtime Data Stream** to change the data pipeline Input source.

2. Select a different value for an existing Category, if you are keeping the existing Input **Data Source** or **Realtime Data Stream**.

3. Click **X** to delete a data source category and value.

4. Click **Add new** to add another **Category** and **Value**. Continue adding as many as you have defined.

5. Select a different category.

**Transformation - Edit a Function**
------------------------------------

You can do any of the following tasks.

1. Select a different **Function**.

2. Add or update the **Sampling Interval** for any new or existing function.

   1. If not selected, create a **Sampling Interval** by selecting **Enable**

   2. Enter a interval value.

   3. Select an interval: **Millisecond**, **Second**, **Minute**, **Hour**, or **Day**.

3. Enter a value for any parameters defined in the function.

4. If no parameters are defined, this field's name is shown as parens characters. **( )**

5. [Optional] Click the plus sign (**+**) to add another function.

6. This step is useful if you want to chain functions together, feeding transformed data from one function to another, and so on.

7. You can also create a function here by clicking **Upload**, which displays the **Add Function** page.

   4.  **Name** your function. Up to 200 alphanumeric characters are allowed.

   5.  **Description**. Provide a relevant description for the function.

   6.  **Project**. Select the project you first selected for this data pipeline.

   7.  **Language**. Select the scripting language for the function: golang, python, node.

   8.  **Runtime Environment**. Select the runtime environment for the function. A default runtime environment is already selected in most cases depending on the **Language** you have selected.

   9.  Click **Next**.

   10. Click **Choose File** to navigate to a file containing your function code.

   11. After uploading the file, you can edit the file contents. You can also choose the contrast levels **Normal**, **Dark**, or **High Contrast Dark**.

   12. Click **Add Parameter** to define and use any parameters for the data transformation in the function.

8. Click **Create**.

**Output - Edit a Destination**
-------------------------------

You can do any of the following tasks.

1. Select a **Edge** or **CloudDestination** to specify where the transformed data is to be output.

2. If you select **Destination** > **Edge**:

   1. **Endpoint Type**. Select **MQTT** or **Realtime Data Stream**.

   2. **Name** your destination. Up to 200 alphanumeric characters are allowed.

   3. **Note:** Elasticsearch and Kafka are available for tech preview use as edge endpoint types in data pipelines. Do not use tech preview features in production environments.

3. If you select **Destination** > **Cloud**, select a **Cloud Type**.

4. if you select Cloud Type as **AWS**:

   4. **Cloud Profile**. Select your existing profile.

   5. See the topic Create Your Cloud Profile in the `Xi IoT Administration Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Xi-IoT-Infra-Admin-Guide:Xi-IoT-Infra-Admin-Guide>`__.

   6. **Endpoint Type**. Select an existing endpoint type.

   7. **Endpoint Name**. Name your endpoint. Up to 200 alphanumeric characters and the dash character (-) are allowed.

   8. **Region**. Select an AWS region.

5. If you select **Cloud Type** as **GCP**:

   9.  **Cloud Profile**. Select your existing profile.

   10. **Endpoint Type**. Select an existing endpoint type.

   11. **Endpoint Name**. Name your endpoint. Up to 200 alphanumeric characters and the dash character (-) are allowed.

   12. **Region**. Select a GCP region.

6. Click **Update**.

..

   **Note:** See `Appendix <#required-cloud-connector-permissions>`__
   for external permissions required to publish data vis public cloud
   connectors.

**Removing a Data Pipeline**
----------------------------

This topic describes how to delete a data pipeline and assumes you have
already logged on to the Xi IoT management console.

1. Click the menu button, then click **Apps and Data** > **Data Pipelines**.

2. Select a data pipeline, click **Remove**, then click **Remove** again to confirm.

**Appendix**
============

**Required Cloud Connector Permissions**
----------------------------------------

Xi IoT requires the following permissions from each service to publish output data.

AWS S3

   s3:ListBucket: Needed for listing of existing buckets and for HEAD Bucket operation.

   s3:CreateBucket: *Needed for bucket create operation if the bucket is not already present.*

   s3:PutObject: Needed to write objects to S3 buckets.

   Check the `S3 Permissions <https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.htm>`__ page for more details on the permissions and related actions.

AWS Kinesis

   stream:DescribeStream: Needed for checking if the Kinesis Data Stream exists and is active before attempting to write records.

   stream:CreateStream: *Needed for creating a Kinesis Data Stream if it does not already exist.*

   stream:PutRecord: Need for writing records to Kinesis Data Streams.

   stream:PutRecords: Need for writing a batch of records to Kinesis Data Streams.

   Check the `Kinesis Permissions <https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonkinesis.html>`__ page for more details on the permissions and related actions.

AWS SQS

   sqs:ListQueues: Needed for checking if the Queue already exists.

   sqs:CreateQueue: Needed for creating a Queue before writing to it.

   sqs:SendMessage: Needed for sending messages to a Queue.

   sqs:SendMessageBatch: Needed for sending a batch of messages to a Queue.

   Check the `SQS Permissions <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-api-permissions-reference.html>`__  page for more details on the permissions and related actions.









Welcome
#######

Welcome to the Xi IoT - Data Pipelines - Getting Started Guide, v0.2.

.. _getting_started:

What We Are Doing
#################
  
The Nutanix Python API Lab will cover a couple of key points.

- Creation of a simple Python Flask web application.
- Creation of a single basic view to display cluster details for the user.
- A backend model to talk to the Nutanix APIs.
- JavaScript to create the interface between the front- and back-end parts of the application.

.. _requirements:

What We Aren't Doing
####################

This lab is not intended as a guide that can be used to learn Python development.  While the copy & paste steps will allow you to create a working application, previous experience with Python will aid you in understanding what each section does.

However, the lab will include links to valuable explanation and learning resources that can be used at any time for more information on each section.  For example, the general structure of this application is almost identical to the one provided by the official Python Flask tutorial and will often link to resources there.

Requirements
############

To successfully complete this lab, you will need an environment that meets the following specifications.

- Previous experience with Python is recommended but not strictly mandatory
- An installation of Python 3.6 or later.  For OS-specific information, please see the next section.
- Python `pip` for Python 3.6.
- Python Flask.  On **most** systems this should be case of running `pip3 install flask`.   On Windows, `pip3.exe` is in the `Scripts` folder within the Python install location.
- The text editor of your choice.  A good suggestion is Visual Studio Code_ as it is free and supports Python development via plugin.
- cURL
- cURL (for Windows - see below).

.. _code: https://code.visualstudio.com/

Python 3.6 on OS X
------------------

- Install Python 3.6 on OS X by downloading the OS X installation package_.

.. _package: https://www.python.org/downloads/mac-osx/

Python 3.6 on Ubuntu 18.04
--------------------------

From the terminal, the following commands can be used to install Python 3.6:

.. code-block:: bash

    sudo apt-get -y update
    sudo apt-get -y install curl
    sudo apt-get -y install python3-dev python3-pip
    sudo apt-get -y install python3-venv
    sudo apt-get -y install python3-setuptools

Python 3.6 on CentOS 7
----------------------

From the terminal, the following commands can be used to install Python 3.6:

.. code-block:: bash

    sudo yum -y update
    sudo yum -y install curl
    sudo yum -y install epel-release
    sudo yum -y install python36
    python3.6 -m ensurepip
    sudo yum -y install python36-setuptools

Python 3.6 and cURL on Windows
------------------------------

- Install Python 3.6 by downloading the Python 3.6 installer_.
- Install cURL from the cURL website_.

.. _installer: https://www.python.org/downloads/release/python-360/
.. _website: https://curl.haxx.se/windows/

.. note::

  If you are running through this lab using Nutanix Frame, Python 3.6 has been installed in the c:\python36 directory.  cURL has also been installed in the c:\tools directory.

Note that cURL is not required to create the demo app.  cURL command samples are provided throughout the lab and may be used for reference at any time.  This is due to its cross-platform nature vs supporting platform-specific commands (e.g. PowerShell).

**Note re Windows systems:** As at January 2019, a **default** installation of Python 3.6 will be installed in the following folder:

.. code-block:: bash

    C:\Users\<username>\AppData\Local\Programs\Python\Python36

.. _optional_components:

Optional Components
###################

In addition to the requirement components above, the following things are "nice to have".  They are not mandatory for these labs.

- A Github account.  This can be created by signing up directly through GitHub_.
- The GitHub Desktop_ application (available for Windows and Mac only)
- Postman_, one of the most popular API testing tools available.

.. _GitHub: https://github.com/
.. _Desktop: https://desktop.github.com/
.. _Postman: https://www.getpostman.com/

.. _cluster_details:

Cluster Details
###############

In a presenter-led environment you will be using a shared Nutanix cluster.  Please use this cluster when carrying out your cURL and application testing.

In a self-paced environment you will need access to a Nutanix cluster along with the credentials required to access it.
