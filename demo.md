# Guided workflow to generate VA extensions

In this demonstration example, we show that users can automatically create an Alexa skill for querying well-known KGs, such as DBpedia.
We see how to exploit the **generator configuration** and the **generator** components,
we discuss the required **steps to upload the skill** on Alexa service providers,
and we demonstrate the **skill in practice** by posing questions on Alexa Developer Console. 

The demo is also available as a [guided video](https://drive.google.com/file/d/1zvWgcO2FeHTNgnFVPWN2pra221oTKYKH/view?usp=sharing).

## The generator configuration component

To avoid manually creating the generator component configuration file, we provide users with a *generator configuration* component.
It takes as input the SPARQL endpoint of interest (and the named graph if any). Moreover, users can customize the skill name (param -i which stands for *invocation name*).

```js
python generator_configuration.py -e http://dbpedia.org/sparql -g http://dbpedia.org -i "dbpedia demo"
```
It automatically retrieves both classes and relations labels and their URIs, and returns the configuration file that can be directly used to initialize the VA generator.
As an alternative, the returned configuration file can be manually inspected and revised. For instance, we noticed that by defaul the *label* property were attached to dbp:label(s) and we changed it with rdfs:label.

## The generator component

1. The generator takes as input a configuration file (in JSON format) that defines 
- the invocation name (i.e., the skill wake-up word),
- the SPARQL endpoint of interest,
- the list of desired intents,
- the VA extension language (*en* and *it* are supported at the moment, but the language management mechanism is designed to be easily extendend with further languages),
- the number of listed result in the reply,
- the specification of a (complete or partial) set of values that entities and properties can assume as a dictionary

```json
{
    "invocation_name": "dbpedia demo", 
    "endpoint": "http://dbpedia.org/sparql",
    "intents": [
        "getAllResultsPreviousQuery",
        "getQueryExplanation",
        "getFurtherDetails",
        "getPropertyObject",
        "getDescription",
        "getNumericFilter",
        "getNumericFilterByClass",
        "getClassInstances",
        "getTripleVerification",
        "getLocation",
        "getSuperlative",
        "getPropertySubjectByClass",
        "getPropertySubject"
    ],
    "lang": "en",
    "result_limit": 5, 
    "entity": {
	<label> : [<URL>, ...],
        "synonyms" : [<synonym>, ...]
    },
    "property": {...}
}
```
2. Users can run the generator
```js
python va-generator.py
```

It returns a folder named as the *invocation name*, dbpedia_demo in our case, which contains
- the *interaction\_model.json* file containing the supported intents and the related utterances, and (partial or complete) set of entities and relations slot values.
- the *back\_end.zip* modelling the skill logic.

## Skill upload on Alexa service providers

Users have to upload the skill interaction model on Amazon Developer and the back-end implementation on AWS by following the **standard** procedure.

### Interaction model on Amazon Developer

You can follow a step by step procedure on the [official Alexa documentation](https://developer.amazon.com/en-US/docs/alexa/devconsole/create-a-skill-and-choose-the-interaction-model.html#create-a-new-skill).
When users have to perform the *upload the generated interaction model* step, they can directly import the auto-generated interaction\_model.json returned by the generator.

### Lampda function on AWS

To configure you lambda function as skill back-end logic, users should perform the following steps:
- create a lambda function by following a step by step procedure on the [official Alexa documentation](https://developer.amazon.com/en-US/docs/alexa/custom-skills/host-a-custom-skill-as-an-aws-lambda-function.html#deploy-app);
- [connect the Lambda function to your skill](https://developer.amazon.com/en-US/docs/alexa/custom-skills/host-a-custom-skill-as-an-aws-lambda-function.html#connect-to-skill);
- [configure the trigger for a Lambda function](https://developer.amazon.com/en-US/docs/alexa/custom-skills/host-a-custom-skill-as-an-aws-lambda-function.html#configuring-the-alexa-skills-kit-trigger);
- upload the auto-generated back-end implementation returned by the generator as a zip file.

## Skill in practice

The DBpedia skill can reply to questions like 
- *Who is the creator of goofy? How tall is Michael Jordan? Can you define Madama Butterfly?*
 to retrieve the object value of a KG triple; 
- *How many programming languages are there?* as a special case of class refinement; 
- *Which movie has producer equals to Hal Roach?* to retrieve the subject of KG triples; 
- *Which library has established before 1400?* representing of a numeric filter; 
- *Which is the river with maximum length?* modeling superlatives; 
- *Can you check if Goofy has Walt Disney as creator?* representing an ask query.

