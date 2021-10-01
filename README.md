

# __HBMQTT 💜 Paho 💙 MySQL + MongoDB__

## __1. HBMQTT & Paho__


- Install packages:
    
    ```bash
    $ pip install hbmqtt paho-mqtt asyncio
    ```

- Create an __*MQTT broker*__ (_broker.py_):

    ```python
    import logging
    import asyncio
    from hbmqtt.broker import Broker
    from hbmqtt.client import MQTTClient, ClientException
    from hbmqtt.mqtt.constants import QOS_1

    logger = logging.getLogger(__name__)

    config = {
        'listeners': {
            'default': {
                'type': 'tcp',
                'bind': 'localhost:9999'    # 0.0.0.0:1883
            }
        },
        'sys_interval': 10,
        'topic-check': {
            'enabled': False
        }
    }

    broker = Broker(config)

    @asyncio.coroutine
    def startBroker():
        yield from broker.start()

    @asyncio.coroutine
    def brokerGetMessage():
        C = MQTTClient()
        yield from C.connect('mqtt://localhost:9999/')
        yield from C.subscribe([
            ("LINTANGtopic/test", QOS_1)
        ])
        logger.info('Subscribed!')
        try:
            for i in range(1,100):
                message = yield from C.deliver_message()
                packet = message.publish_packet
                print(packet.payload.data.decode('utf-8'))
        except ClientException as ce:
            logger.error("Client exception : %s" % ce)

    if __name__ == '__main__':
        formatter = "[%(asctime)s] :: %(levelname)s :: %(name)s :: %(message)s"
        logging.basicConfig(level=logging.INFO, format=formatter)
        asyncio.get_event_loop().run_until_complete(startBroker())
        asyncio.get_event_loop().run_until_complete(brokerGetMessage())
        asyncio.get_event_loop().run_forever()
    ```

- Create an __*MQTT subscriber*__ (_sub.py_):

    ```python
    # subscriber
    import paho.mqtt.client as mqtt

    client = mqtt.Client()
    client.connect('localhost', 9999)

    def on_connect(client, userdata, flags, rc):
        print("Connected to a broker!")
        client.subscribe("LINTANGtopic/test")

    def on_message(client, userdata, message):
        print(message.payload.decode())

    while True:
        client.on_connect = on_connect
        client.on_message = on_message
        client.loop_forever()
    ```

- Create an __*MQTT publisher*__ (_pub.py_):

    ```python
    # publisher
    import paho.mqtt.client as mqtt

    client = mqtt.Client()
    client.connect('localhost', 9999)

    while True:
        client.publish("LINTANGtopic/test", input('Message : '))
    ```
#

## __2. HBMQTT, Paho & MySQL__

- Create a database & table on MySQL:
    
    ```bash
    $ create database mqttpy;
    $ use mqttpy
    $ create table(
        id int not null auto_increment,
        message varchar(255),
        time timestamp default current_timestamp,
        primary key (id)
    );
    $ describe mqttpy
    ```

- Install _pymysql_:

    ```bash
    $ pip install pymysql
    ```

- Create an __*MQTT broker*__ (_brokerMySQL.py_):

    ```python
    import logging
    import asyncio
    from hbmqtt.broker import Broker
    from hbmqtt.client import MQTTClient, ClientException
    from hbmqtt.mqtt.constants import QOS_1
    import pymysql

    logger = logging.getLogger(__name__)

    config = {
        'listeners': {
            'default': {
                'type': 'tcp',
                'bind': 'localhost:9999'    # 0.0.0.0:1883
            }
        },
        'sys_interval': 10,
        'topic-check': {
            'enabled': False
        }
    }

    broker = Broker(config)

    @asyncio.coroutine
    def startBroker():
        yield from broker.start()

    @asyncio.coroutine
    def brokerGetMessage():
        C = MQTTClient()
        yield from C.connect('mqtt://localhost:9999/')
        yield from C.subscribe([
            ("LINTANGtopic/test", QOS_1)
        ])
        logger.info('Subscribed!')
        try:
            for i in range(1,100):
                message = yield from C.deliver_message()
                packet = message.publish_packet
                print(packet.payload.data.decode('utf-8'))
                
                con = pymysql.connect(
                    host = 'localhost',
                    user = 'lintang',
                    password = '12345',
                    db = 'mqttpy',
                    cursorclass = pymysql.cursors.DictCursor
                )
                kursor = con.cursor()
                sql = '''insert into mqttpy (message) values (%s)'''
                val = str(packet.payload.data.decode('utf-8'))
                kursor.execute(sql, val)
                con.commit()
                print(kursor.rowcount, 'Data saved!')

        except ClientException as ce:
            logger.error("Client exception : %s" % ce)

    if __name__ == '__main__':
        formatter = "[%(asctime)s] :: %(levelname)s :: %(name)s :: %(message)s"
        logging.basicConfig(level=logging.INFO, format=formatter)
        asyncio.get_event_loop().run_until_complete(startBroker())
        asyncio.get_event_loop().run_until_complete(brokerGetMessage())
        asyncio.get_event_loop().run_forever()
    ```

#

## __3. HBMQTT, Paho & MongoDB__


- Create a database & collection on MongoDB:
    
    ```bash
    $ use mqttpy
    $ db.createUser({
        'user': 'lintang',
        'pwd': '12345',
        'roles': ['readWrite', 'dbAdmin']
    })
    $ db.createCollection('mqttpy')
    ```

- Install _pymongo_:

    ```bash
    $ pip pymongo
    ```

- Create an __*MQTT broker*__ (_brokerMongoDB.js_):

    ```python
    import logging
    import asyncio
    from hbmqtt.broker import Broker
    from hbmqtt.client import MQTTClient, ClientException
    from hbmqtt.mqtt.constants import QOS_1
    import pymongo

    mymongo = pymongo.MongoClient('mongodb://localhost:27017')
    logger = logging.getLogger(__name__)

    config = {
        'listeners': {
            'default': {
                'type': 'tcp',
                'bind': 'localhost:9999'    # 0.0.0.0:1883
            }
        },
        'sys_interval': 10,
        'topic-check': {
            'enabled': False
        }
    }

    broker = Broker(config)

    @asyncio.coroutine
    def startBroker():
        yield from broker.start()

    @asyncio.coroutine
    def brokerGetMessage():
        C = MQTTClient()
        yield from C.connect('mqtt://localhost:9999/')
        yield from C.subscribe([
            ("LINTANGtopic/test", QOS_1)
        ])
        logger.info('Subscribed!')
        try:
            for i in range(1,100):
                message = yield from C.deliver_message()
                packet = message.publish_packet
                print(packet.payload.data.decode('utf-8'))

                mydb = mymongo['mqttpy']
                mycol = mydb['mqttpy']
                mydata = {"message": packet.payload.data.decode('utf-8')}
                x = mycol.insert_one(mydata)
                print('Data saved!')

        except ClientException as ce:
            logger.error("Client exception : %s" % ce)

    if __name__ == '__main__':
        formatter = "[%(asctime)s] :: %(levelname)s :: %(name)s :: %(message)s"
        logging.basicConfig(level=logging.INFO, format=formatter)
        asyncio.get_event_loop().run_until_complete(startBroker())
        asyncio.get_event_loop().run_until_complete(brokerGetMessage())
        asyncio.get_event_loop().run_forever()
    ```

#

#### RameshSankoju :love_letter: visitrameshs@gmail.com

 
 
[LinkedIn](https://www.linkedin.com/in/ramesh-sankoju-5ba78945/) |
 
:octocat: [GitHub](https://github.com/Ramesh-pythonista) |
 