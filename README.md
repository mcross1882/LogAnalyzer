LogAnalyzer
===========

Log analyzer and summary builder written in Scala built for JVM projects


### Overview

Log analyzer uses a set of analyzer, parsers, and services to parse logfiles into condensed summaries. To understand this better
lets look at each piece individually..

#### Condense log files

Convert <a href="https://github.com/mcross1882/LogAnalyzer/tree/master/examples/sample.txt">bloated log files</a> into clean and concise summaries...

```
Demo Diagnostics Report
=======================
Emergency
- Whoops something went wrong!: 2

FailedLogin
someone@test.com experienced a failed login: 1

DebuggingNoise
Suppressed: 25
```

#### Analyzers

Analyzers are a set of rules that match a regex expression against a single line that is read in from the logfile. They also store simple meta
data like extracted regex text, counters, and the category to which the analyzer belongs too. Analyzers are written in XML (see sample below)
and are located in `src/main/resources/`.

```xml
<analyzers>
    <analyzer category="Emergency" regex="Emergency: (.*)" vars="message">
        - $message
    </analyzer>
    
    <analyzer category="FailedLogin" regex="login (\w+) for (\w+@\w+.\w+)" vars="status|email">
        $email experienced a $status login
    </analyzer>
    
    <analyzer category="DebuggingNoise" regex="Debug: (.*)">
        Suppressed
    </analyzer>
</analyzers>
```


#### Parsers

Parsers are simply a collection of analyzers. They are reusable building blocks that define the types of parsing that will be done on the
input files.

```xml
<parsers>
    <parser name="DemoParser" type="SimpleParser">
        <analyzer category="Emergency" />
        <analyzer category="FailedLogin" />
        <analyzer category="DebuggingNoise" />
    </parser>
</parsers>
```


#### Services

Services represent an application or service that you want to perform log analysis on (e.g. httpd, php, etc...). They contain a list of files
that should be parsed as well as a list of parsers that should be used on the input.

```xml
<services>
    <service name="demo" title="Demo Diagnostics Report">
        <logfiles>
            <file src="examples/sample.txt" />
        </logfiles>
        
        <parsers>
            <parser name="DemoParser" />
        </parsers>
    </service>
</services>
```

### Extending the API

Need a custom parser or analyzer implementation? No problem simply extend the respective trait (e.g. `Parser`, `Analyzer`, or `Service`) and
set the `type` attribute within the XML files to use your newly created class.
