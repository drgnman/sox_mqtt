# mqtt_sox
git cloneして使ってください

```
pip install -r requirements.txt
```

sox_mqtt.pyをimportすることでmqttのpublishとsubscribeを支援します。

## Connection
Connectionオブジェクトを用いて接続するブローカへの設定を行う
connectメソッドによって設定した接続情報の入力確認を行い、接続に成功するとmqtt_clientを戻り値で返す
```
connection = Connection(broker_name, port, client_id=randomId, username=None, password=None)
client = connection.connect()
```


## Create
メタノードとしてそのトピックが扱う情報をretainerに登録します。
メタノード名はpublishの宛先となるノード名_metaとして定義されます。

```python:create.py
publisher = PublishModule(client) # publishモジュールを通す
node = Node(node_name)
node.setLocation(lat, lng)
node.setDescription(description_contents)
```

``` python:create.py
transducer = node.Transducer(transducer_name)
transducer.setUnit(unit)
transducer.setMinValue(int(min_value))
transducer.setMaxValue(int(max_value))
transducer.setDescription(description_contens)
node.appendTransducer(transducer)
```

ノードに情報を設定した後にcreateメソッドで作成する
``` python
publisher.create(node)
```


## Publish
パブリッシャはNodeオブジェクトにデータをセットした上で、publishメソッドを用いてデータを配信します

``` python:publisher.py
publisher = PublishModule(client)
node = Node(node_name)
transducer = node.Transducer(transducer_name)
transducer.setRawValue(value) # 値を設定する
node.appendTransducer(transducer)
```

``` python:publisher.py
publisher.publish(node)
```

## Subscribe
サブスクライバはsubscribeメソッドでnode_nameを指定してサブスクライブ登録をする
SubscribeModuleを継承したオリジナルクラスを定義し、setProcessOnMessageメソッドでデータ受信時の処理を定義する。

``` python:subscriber.py
# サブスクライブした時の処理をOverrideする方法
class OriginalSubscribeModule(SubscribeModule):
    def __init__(self, client):
        super().__init__(client)

    def setProcessOnMessage(self):
        def on_message(client, userdata, msg):
            # データ受信時の処理内容を実装する
        self.client.on_message = on_message
```

subscribe時にはqosを`0`,`1`,`2`から設定可能
デフォルト値は`0`

``` python:subscriber.py
subscriber = OriginalSubscribeModule(client) # 独自クラスを作った場合
subscriber.subscribe(node_name, qos=0)   # subscribeするノードを指定する
subscriber.setProcessOnMessage()  # データ受信時の処理を設定
subscriber.run()                  # サブスクライブ開始
```

## Node
|要素名|説明|
|----|----|
|node_name|ノード名|
|location|位置情報(緯度,経度)の順で指定|
|transducers|ノードが扱うtransducer群|
|description|備考|

## Transducer
|要素名|説明|
|----|----|
|transducer_name|Transducer名|
|unit|単位|
|min_value|想定される最小値|
|max_value|想定される最大値|
|description|備考|
