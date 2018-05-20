# OpenFastTrace Live Demo (Medium)

## Preparation

### Precoditions
The following software is needed to prepare and run this live demonstration

* bash
* maven
* Java 8 JRE (or JDK)
* GNU grep
* GNU sed
* Text editor (e.g. vi)

#### On Debian / Ubuntu

```bash
sudo apt install openjdk-8-jre vim libxml2-utils
```

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

### Specification Items vs. Requirements
In OFT we mostly talk about "Specification Item" which are a superset of requirements. The reason for this is that we want to treat markers in the code in the same way as requirements in a document when it comes to tracing. We will see what that means in a few minutes.

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

If there are no errors, the trace comes back with only a summary in the default verbosity level.

### Finding Errors in the Tracing Chain
Let's intentionally brake the tracing chain.

We will increment the version number of the system requirement we saw earlier. Now the lower level requirements do not cover the system requirement anymore, so we expect OFT to complain about this.

```bash
vi doc/system_requirements.md
/req\~specification-item
```

*(Add a field "Stakeholder (optional)" for the sake of demonstration.)*

We now run the trace again.

```bash
oft trace doc src/main/java src/test/java
```

And sure enough OFT points out that a few links are now outdated.

*(Walk the audience through the trace result. Explain what the symbols in front of the links mean. Take your time - this is one of the fundamental aspects of OFT.)*

### Fixing an Error
We are now going to point the design specification item to the new version of the system requirement and in turn increment the version number of the design item.

```bash
/req\~specification-item
```

*(Increment the version number of links and linking items. Add a field "Stakeholder (String, optional)". Note that there are multiple design items that need to be updated!)*

Again, we run the trace.

```bash
oft trace doc src/main/java src/test/java
```

Now the problem is pushed down a level from the design to the implementation.

### Tracing Upward Only
Let's assume you are responsible for OFT's design document and someone else has the job to implement what you designed.

In this case you only care about if you covered the system requirements properly.

Next we are going to run a trace that focuses on this question only. We do this by filtering out all artifact types but `feat`, `req` and `dsn`. The first two are from the system requirements you need to cover, the last is what you are responsible for

```bash
oft trace -a feat,req,dsn doc src/main/java src/test/java
```

As you can see at least from the perspective of the design document. While this doesn't mean the whole projects trace is green it at least proves that you did your job.

### Controlling the Output
What if we want to see all specification items in the trace and not only the ones that failed?

```bash
oft trace -v ALL doc src/main/java src/test/java
```

And if we want to find a specific requirement in that trace?

```
oft trace -v ALL doc src/main/java src/test/java | grep '^[^|].*req~tracing.deep-coverage~1'
```

One of the reasons OFT features a plain text report and has a command line interface is that this combination is ideal for chaining OFT with other populare command line tools like `grep`.

Even if OFT tried, it could never cover all use cases that you might come up with in your real world project. Instead of trying to be the jack of all trades OFT focuses on being easy to integrate with other tools.

**Do one thing. Do it well.**

Sounds familiar?

Let's take this one step further an filter out a 

```bash
id='req~tracing.deep-coverage~1' && oft trace -v ALL doc src/main/java src/test/java | grep -Pzo 'ok.*'"$id"'.*\n([|].*\n)+'
```
If you think now: this command is madness - fair enough. The point is that through chaining OFT with other tools you can do things the designers of OFT didn't think of.

### Converting Specification Formats
While the importer of OFT automatically detects the format you feed into a trace, you might need to deliver specifications in a certain format to another organization.

OFT can work as converter. In this case the original content is converted into the target format of your choice.

The following command converts the system requirements and design of OFT into the ReqM2 format.

```bash
oft convert -o specobject doc
```

*(The next step assumes you have `xmllint` installed. Of course you can use other software to pretty print the XML output too)*

```bash
oft convert -o specobject doc | xmllint --format - > /tmp/oft_specifications.xml
```

```bash
vi /tmp/oft_specifications.xml
```

### Which Exchange Formats Does OFT support?

* OFT native (in / out)
* ReqM2 (in / out)
* OFT code decoration (in)
* Reqm2 code decoration (in)

Planned:

* RIFF (in)
* CSV (out)

### Where to go From Here?

You can find documentation about OpenFastTrace on GitHub:

* [OpenFastTrace project page](https://github.com/itsallcode/openfasttrace)
* [Contributing to OFT](https://github.com/itsallcode/openfasttrace/CONTRIBUTING.md)

Handbook is work in progress (at the time of writing this script 2018-05-20):
* [User guide](https://github.com/itsallcode/openfasttrace/blob/story/%23111-Add_handbook/doc/user_guide.md)
