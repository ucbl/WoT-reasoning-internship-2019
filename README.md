# WoT Reasoning

The aim of this project was to demonstrate how to add reasoning capabilities to [Web of Things](https://www.w3.org/WoT/) application platforms. Currently, it can infer properties so as to discover devices that have not been formally declared in a WoT Runtime, thanks to *ad hoc* reasoning rules.

This project is based on the following technologies:

- [node-red](https://nodered.org/) and its upper-layer [iot-schema-node-red](https://github.com/iot-schema-collab/iotschema-node-red) as WoT application programming environment,
- [Thingweb Directory](https://github.com/thingweb/thingweb-directory) as support for WoT device descriptions in W3C [Thing Description](https://w3c.github.io/wot-thing-description) vocabulary, and
- [GraphDB](https://www.ontotext.com/products/graphdb) as both persistence layer for device semantic descriptions and semantic reasoner.

## Getting started

### node-red :  

- Installation: `sudo npm install -g --unsafe-perm node-red`
- Launching: `node-red`

### GraphDB  

- Installation (8.10.1 recommended; requires Java 8): `https://www.ontotext.com/products/graphdb/graphdb-free/`
- Launching:
  1. `cd graphdb-free-8.10.1/bin`
  2. `./graphdb -d`

GraphDB Web interface will be accessible at `http://localhost:7200`  

### ThingWeb Directory

- Installation (requires a particular version of WoT TD): `git clone https://github.com/thingweb/thingweb-directory/tree/4641f260190805034ed11c11b0591280fca06d1b`
- Launching:
  1. set environment variables to match thingweb and GraphDB:  
    `export THINGWEB_SPARQL_QUERY_ENDPOINT=http://localhost:7200/repositories/<your_repository_name>`  
    `export THINGWEB_SPARQL_UPDATE_ENDPOINT=http://localhost:7200/repositories/<your_repository_name>/statements`
  2. `cd thingweb-directory/directory-app`
  3. `gradle run`  

### iotschema-node-red  

- Installation: 
  1. `git clone https://github.com/iot-schema-collab/iotschema-node-red`  
  2. Go into node-red local folder:
     Linux: `$ cd ~/.node-red`
     Windows (Powershell): `PS C:\Users\{username}\AppData\Roaming\npm\node_modules\node-red`
  3. `$ npm i -g recursive-install`
  4. `$ npm-recursive-install --rootDir={YOUR_OWN_PATH}/iotschema-node-red`
- Launching: iotschema-node-red devices and capabilities will be available on node-red interface.

## Repository contents

- `wot-reasoning-example.json` is a basic example of device description (to add in node-red) that contains the definition of a `KelvinTemperatureDevice`.
- `builtin_owl2-rl_modified.pie` is the ruleset to add when creating a new RDF repository in GraphDB. It contains:
  - the definition of OWL2RL semantics,
  - the definition, as a set of rules, of a second device: `kelvin_to_celsius`, which performs temperature unit conversion.
  - some specific inference rules (at the end of the file) defining a property chain to be used with the example below,

**Note**: it is of course possible to define the `kelvin_to_celsius` convertor in the TD json file instead of the GraphDB ruleset.

## Using the example

Basically, based on the definitions of a `KelvinTemperatureDevice` and a `kelvin_to_celsius` converter, this example infers a `CelsiusTemperatureDevice` which is queriable in SPARQL.

### node-red side

  * Open `wot-reasoning-example.json` in node-red and on the *thing-directory* tab, choose the *Use Remote TD Server* option then type `localhost:8080` (*i.e.* the adress used for thingweb)
  * Store the TD
  * The recipe will be automatically redirected to GraphDB

### GraphDB side

The GraphDB SPARQL endpoint should expose both the *KelvinTemperatureSensor* node **and** an implicit node called *CelsiusTemperatureSensor* inferred by the reasoner (if activated in the project settings) using the provided ruleset.  

You should be able to query both nodes of type "iot:TemperatureSensor" using:
```SQL
SELECT * WHERE {
    ?s iot:capability "iot:TemperatureSensor".  
    ?s ?p ?o.
}
```  

This query shows that the reasoner has created a *KelvinTemperatureSensor* (i.e. a device that produces temperature observation results in Kelvin).

**Note**: the rule that generates this new device is an *ad hoc* rule. It still needs to be expressed using an OWL property chain axiom to be generalized and more useful.

## Troubleshooting

- EACCESS problem when starting node-red -> `$ sudo npm config set unsafe-perm=true`

## Credits

This project was realized during a first year master degree internship funded by the computer science department of Université Claude Bernard Lyon 1.