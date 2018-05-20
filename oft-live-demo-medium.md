# OpenFastTrace Live Demo (Medium)

## Preparation

### Precoditions
The following software is needed to prepare and run this live demonstration

* bash
* maven
* Java 8 JRE (or JDK)
* GNU grep
* GNU sed

### Preparing the Directory That Contains OpenFastTrace
Run the following preparation ahead of your presentation so that you can dive in right away.

```bash
version='0.5.3'
demodir="$HOME/tmp/openfasttrace-demo-medium"
mkdir -p "$demodir"
cd "$demodir"
git clone -b "$version" https://github.com/itsallcode/openfasttrace
cd openfastrace
mvn package
alias oft="java -jar $demodir/openfasttrace/target/openfasttrace-$version.jar"
```

After that you will have a clone of the OFT repository in a specific version detached-head-mode.

Choosing a specific version is important to avoid surprises during the presentation in case OFT's behavior changed since this live demo script was written.

## Holding the Live Demonstration
The following sections contain a suggestion of what you can tell people interested in OFT about the project and what it is good for. As with every presentation it is better to memorize the general sequence instead of the wording. You are not reciting a poem, feel free to use your own words!

## Live Demo

### What is OFT?
OpenFastTrace is a requirement tracing suite. Simply put requirement tracing makes sure that you don't forget stuff in your software project. That means that if you use OFT, you should neither run into a situation where you have to explain to your customers why you forgot to implement a feature. 

On the other hand OFT will tell you if there is code in your project where the original requirement has long since been rejected, allowing you to clean up your code base and thus keep it maintainable and more secure.

OFT achieves this by crawling your specification documents and code and correlating them with the help of special IDs.

### Requirements in OFT
Let's have a look at the OFT system requirement specification to get an idea of how those ID look and in what context we use them.

```bash
grep -B 1 -A 33 'req~specification-item~' doc/system_requirements.md
```

You see an excerpt of a specification document written in Markdown. More precisely you see a requirement with a title, a requirement ID and a description.

Incidentally I picked a the requirement that defines a "Specification Item".

*(Go through the fields and explain them shortly to your audience.)*

This requirement states that it wants to be covered by a design.

### Links to the Design
You saw that the requirement in the system specification requires coverage in a design document. The easiest way for us to find this coverage - apart from using OFT - is to use a simple text search.

```bash
grep -n 'req~specification-item~' doc/design.md
```

As you can see we get multiple matches. Each match is part of a design requirement that references the system requirement we saw earlier.

```bash
grep -B 20 -A 3 'req~specification-item~' doc/design.md
```

### Code Decorations
In order to be able to tell where in your code a requirement is covered, you need to decorate your code with special tags contained in comments.


```
grep -rnA 3 '~specification-item~' src | sed -e 's/java.*trace/.../'
```

You see a number of matches under `src/main/java` and `src/test/java`. As you might imagine those are the places where the design requirements are implemented and tested.

### OFT Modes
OFT has two modes of operation:
1. Tracer
2. Format converter

In this live demonstration we will focus mostly on the tracer since this is the core functionality.

While the converter is undoubtedly useful, it simply converts all its inputs into a specific requirements interchange format.

### Tracing OFT With Itself
You might have heard the term "eat your own dogfood". This means that an organization should use its own products in order to share the customer's experience.

Of course we do this with OpenFastTrace. So lets run a trace over our own sources.

### Where to Find OFT
