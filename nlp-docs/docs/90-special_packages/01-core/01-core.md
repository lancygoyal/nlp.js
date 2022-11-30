import GithubLink from '@site/src/components/GithubLink';

# The module

## Installation

You can install @nlpjs/core:

```bash
    npm install @nlpjs/core
```

## Container

The Container is an IoC container class. Basically it's able to store instances and classes, marked as being singleton or not.
When a class is registered as a singleton, a new instance is created and everytime that the container is asked for this class the same instance is returned. Same behaviour if we register an instance as singleton, but in this case this instance is returned always.
When an instance is registerd as not singleton, the class of the instance is stored and when asked to the container it returns a new instance of this class.

Also in the container can be registered settings with the name of the objects registered, so these settings are provided to the new created instances.

And also pipelines can be registered. Pipelines are sequences of commands that are executed based on the classes and instances known by the container.

### Plugins

At the plugins folder you will find some different classes with atomics tasks:

- _split_: given an object with a property _text_, split this text into an array of characters into the property _splitted_
- _reverse_: given an object with a property _splitted_ that is an array, reverse this array
- _join_: given an object with a property _splitted_ that is an array, join this array into an string into the property _text_
- _lower_: given an object with a property _text_, convert this string to lower case.
- _upperFirst_: given an object with a property _text_, convert the first character to upper case

### Pipelines

If you take a look at the pipelines.md file you'll see this:

```markdown
# Pipelines

## reverse

split
reverse
join
->output.text

## reverse-and-\*

$reverse
lower
->output.text

## reverse-and-capitalize

$super
upperFirst
output.text
```

_reverse_, _reverse-and-$ast_ and _reverse-and-capitalize_ are pipelines. Each one contains commands to be executed in sequence. As you can see, the commands are the name of the plugins, except some exceptions:

- _output.text_: this is telling that at this step take a look at the output object (the one that is returned by the previous step) and from it extract the property text before going to the next step
- _$reverse_: When you start a name with the $ that means not to execute a plugin but to call another pipeline using as input the current output.
- _$super_: This is to call the parent. You can see that _reverse-and-&ast_ have an asterisk at the end, this is a wildchar, so will be the parent of those pipelines that match this pattern, in this case _reverse-and-capitalize_ match this name so if a child pipeline.
- _->output.text_: when a command starts with the _->_ that means that this command should be executed only if this pipeline is the one executed directly, but not when this pipeline is invoked from another pipeline. So when calling _reverse_ the output will be an string, but when calling _reverse_ from _reverse-and&ast_ this last command will be not execute so the output of _reverse_ will be the object and not the string.

### Creating a container

This is the way of creating a bootstrapped container:

```javascript
const { containerBootstrap } = require("@nlpjs/core");
const container = containerBootstrap();
```

This automatically will:

- load the .env file if exists as environment variables
- load the conf.json file if exists as configuration (will be explained in other example).
- If the conf.json file does not exists or it does not includes a pipelines path, then load the ./pipelines.md by default as pipelines
- If the conf.json file does not exists or it does not includes a plugin path, then load the content of the folder ./plugins as plugins.

### Executing a pipeline

Containers have a method _runPipeline_ where you can pass the name of the pipeline (or the compiled pipeline) and an input to be processed.
Important: this input usually travel through each step of the pipeline, so can be modified by the pipeline execution.
This is an example of code calling the pipeline _reverse-and-capitalize_, that shows how to do inheritance of pipelines, call other pipelines, and have commands that are only executed if the depth of the call is 0 (belongs to the called pipeline).

```javascript
const { containerBootstrap } = require("@nlpjs/core");

async function main() {
  const container = containerBootstrap();
  const input = "GNIHTEMOS";
  const result = await container.runPipeline("reverse-and-capitalize", input);
  console.log(result); // It should log "Something"
}

main();
```

## Example of Usage

You'll find an example of usage at <GithubLink to="examples/01-container/README.md">examples/01-container</GithubLink>