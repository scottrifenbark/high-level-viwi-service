# Usage

## JSon configuration file

This file must be named *high.json*, and must accessible from afb-daemon.
The json configuration file consists in 2 sections:

### Definitions section

This section describes each resources defined in the high-level binding. Each resource is composed with different properties having a name, a type and a description.
Type can be boolean, double, string, or int. Properties "id", "uri" and "name" are compulsory.

For instance:

```json
{
	"name": "/car/demoboard/",
	"properties": {
		"id": {
			"type": "string",
			"description": "identifier"
		},
		"uri": {
			"type": "string",
			"description": "object uri"
		},
		"name": {
			"type": "string",
			"description": "name"
		},
		"unit": {
			"type": "string",
			"description": "units"
		},
		"speed": {
			"type": "double",
			"description": "vehicle centerpoint speed as shown by the instrument cluster"
		},
		"rpm": {
			"type": "double",
			"description": "engine rotations per minute"
		},
		"level": {
			"type": "double",
			"description": "level of tankage"
		},
		"load": {
			"type": "double",
			"description": "engine load"
		}
	}
}
```

<!-- pagebreak -->

### Resources section

This section defines which values should be assigned to resource's properties as defined in the definitions section.
The link to the definitions section is made through the name of the resource.

Some values are static, some are linked to low-level requests.

In case a value is linked to a low-level request, the value will start with "${" and end with "}". In that case the value will consist in the name of the low-level signal, followed
with the frequency of the signal in ms. -1 in the frequency means that high level binding should subscribe to low level binding for all changes, without specifying a frequency.

For instance:
```json
{
	"name": "/car/demoboard/",
	"values": [{
		"name": "vehicleSpeed",
		"unit": "km/h",
		"speed": "${diagnostic_messages.vehicle.speed,1000}"
	}, {
		"name": "engineSpeed",
		"unit": "rpm",
		"rpm": "${diagnostic_messages.engine.speed,1000}"
	}, {
		"name": "fuelLevel",
		"unit": "litre",
		"level": "${diagnostic_messages.fuel.level,1000}"
	}, {
		"name": "engineLoad",
		"unit": "Nm",
		"load": "${diagnostic_messages.engine.load,1000}"
	}]
}
```

<!-- pagebreak -->

## Running and testing

### Launch the binder together with the two bindings

```bash
$ cd $WD
$ afb-daemon --rootdir=$WD/low-level-can-service/CAN-binder/build/package --ldpaths=$WD/low-level-can-service/CAN-binder/build/package/lib:$WD/high-level-viwi-service/build/high-can-binding --port=1234 --tracereq=common --token=1 --verbose
```

### Use afb-client-demo to test high level binding

On another terminal, connect to the binding using previously installed _**AFB Websocket CLI**_ tool:

```bash
$ afb-client-demo ws://localhost:1234/api?token=1
```

You will be on an interactive session where you can communicate directly with the binding API.

The binding provides at this moment 3 verbs, _get_, _subscribe_ and _unsubscribe_, which can take a JSON object as an argument.


To use the _**AFB Websocket CLI**_ tool, a command line will be like the following :

```
<api> <verb> <arguments>
```

Where:

* API : _**high-can**_.
* Verb : _**get**_, _**subscribe**_ or _**unsubscribe**_
* Arguments : _**{ "name": "/car/doors/" }**_

You can therefore use commands such as:

```
high-can subscribe {"name":"/car/doors/","interval":10000}
high-can unsubscribe {"name":"/car/doors/","interval":10000}
high-can get {"name":"/car/demoboard/"}
high-can get {"name":"/car/demoboard/","fields":["fuelLevel","engineLoad"]}
```

For instance the output of the third command should be:

```
high-can get {"name":"/car/demoboard/"}
ON-REPLY 1:high-can/get: {"response":{"\/car\/demoboard\/2159e2-5b638a-39e242-7a2f5":{"id":"2159e2-5b638a-39e242-7a2f5","name":"vehicleSpeed","speed":0.000000,"unit":"km\/h","uri":"\/car\/demoboard\/2159e2-5b638a-39e242-7a2f5"},"\/car\/demoboard\/22ad2c-5a3c2b-50fabb-324c82":{"id":"22ad2c-5a3c2b-50fabb-324c82","level":0.000000,"name":"fuelLevel","unit":"litre","uri":"\/car\/demoboard\/22ad2c-5a3c2b-50fabb-324c82"},"\/car\/demoboard\/3a3ab9-2bd52c-11d30-689acf":{"id":"3a3ab9-2bd52c-11d30-689acf","name":"engineSpeed","rpm":0.000000,"unit":"rpm","uri":"\/car\/demoboard\/3a3ab9-2bd52c-11d30-689acf"},"\/car\/demoboard\/5ae808-8093cb-99716-30a605":{"id":"5ae808-8093cb-99716-30a605","load":0.000000,"name":"engineLoad","unit":"Nm","uri":"\/car\/demoboard\/5ae808-8093cb-99716-30a605"}},"jtype":"afb-reply","request":{"status":"success","uuid":"44ce03f9-a7ca-49e1-a62a-40c74db0caa0"}}
```

As you can see for the moment all values are 0, because we didn't inject any CAN data in the binder. To do this, you can use **canplayer** to feed the bindings with some data.
You can find an example of data in high level binding, "samples" directory.

For instance, on a third terminal:

```bash
$ canplayer -I candata
```
