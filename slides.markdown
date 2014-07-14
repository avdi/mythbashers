# MythBashers
### Avdi Grimm



![MythBusters Logo](mythbusters.jpg)


### Jamie Hyneman
![Jamie Hyneman](jamie.jpg)


### Take a common myth ###
![Archimedes Mirror](mirror2.jpg)


### Devise an experiment ###
![Adam planning the test](planning.png)


### Build a test rig ###
![Jamie experimenting with mirrors](building.png)


### Test the myth! ###
![Kids with mirrors](boat.png)


### Make a determination ###

- Confirmed
- Plausible
- Busted


### Cool Toys ###
![Pop Gun](popgun.png)


### Me too! ###


### Ruby ###


### Multi-Disciplinary ###


### Lingua Franca ###


### Shell! ###


### Bash ###


### The myth ###

<p class="fragment">"Bash isn't Webscale"</p>


### The build ###

<p class="fragment">A single-page web chat application, backed by a Bash script</P>


![app screenshot](screenshot.png)
https://github.com/avdi/walrus


### Highlights


## Serving HTTP


### Netcat
Like cat(1) for TCP/UDP sockets.


### A Simple HTTP server
```bash
$ printf "HTTP/1.1 200 OK\r\n\r\nHello world\n" |
    nc -l 8080
```


### Client
```bash
$ curl localhost:8080
Hello world
```


### Netcat output
```bash
$ printf "HTTP/1.1 200 OK\r\n\r\nHello world\n" |
    nc -l 8080
GET / HTTP/1.1
User-Agent: curl/7.32.0
Host: localhost:8080
Accept: */*
```



## Serving Pages Dynamically


### Netcat is Read/Write
No callbacks.


### Dynamic Netcat
Branching on input.


### Coprocesses ###


`coproc NAME COMMAND`


### Rot13 Coprocess
```bash
$ coproc rot13 { stdbuf -oL tr '[A-Za-z]' '[N-ZA-Mn-za-m]'; }
[1] 26812
```


### Coprocess Filehandles
```bash
$ echo ${rot13[0]}
63
$ echo ${rot13[1]}
60
```


### Writing and Reading
```bash
$ echo "hello, world" >&${rot13[1]}
$ echo "this is a secret" >&${rot13[1]}
$ read line <&${rot13[0]}
$ echo $line
uryyb, jbeyq
$ read line <&${rot13[0]}
$ echo $line
guvf vf n frperg
```


### Ending a Coprocess
#### The rude way
```bash
$ kill ${rot13_PID}
```


### Ending a Coprocess
#### The polite way
<div class="fragment current-visible">
<p>Closing a filehandle</p>
<pre data-trim><code data-trim>
somecommand 1&>-
</code></pre>
</div>
<div class="fragment current-visible">
<p>Closing a filehandle without a command</p>
<pre data-trim><code data-trim>
exec 60&>-
</code></pre>
</div>


<p>Interpolating the coprocess STDIN</p>
<pre data-trim><code data-trim>
exec ${rot13[1]}&>-
</code></pre>
<p class="fragment">(Doesn't parse correctly)</p>


<p>Using <code>eval</code></p>
<pre data-trim><code data-trim>
eval "exec ${rot13[1]}&>-"
</code></pre>
<div class="fragment">
<p>Simple, right?</p>
</div>


### Iä! Iä! Cthulhu Fhtagn!
Ph'nglui mglw'nafh Cthulhu R'lyeh wgah'nagl fhtagn!
<img src="cthulhu.gif" class="stretch"/>


### Netcat as a coprocess
  ```bash
  # Start netcat
  coproc nc ( nc -l 8080 )

  # Plug it into a request handler
  handle_request ${nc_PID} <&${nc[0]} >&${nc[1]}

  # Close filehandles
  eval "exec ${nc[1]}>&-"     
  eval "exec ${nc[0]}<&-"
  ```



## Organizing the Program

![Jamie's Shelves](shelves.png)


### Shell Functions ###


### Miniature Shell Scripts ###
- Positional params (`$1`, `$2`, `$*`, etc.)
- Redirectable STDIN, STDOUT, and STDERR
- Return an integer exit status


### Naive ROT13 Function

```bash
rot13() {
  echo "$1" | stdbuf -oL tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
}
```

<div class="fragment">
<pre><code data-trim>
$ rot13 "pssst"
cfffg
</code></pre>
</div>


### Pipeline ROT13 Function

```bash
rot13() {
  stdbuf -oL tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
}
```

<div class="fragment">
<pre><code data-trim>
$ echo "pssst" | rot13
cfffg
</code></pre>
</div>


### stdbuf

```bash
stdbuf -oL tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
```


![An empty pipe](avdi_pipeline_full_white.jpeg)


```bash
stdbuf -oL tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
```


### Local Variables

```bash
foo() {
  bar=42 # global!
}
$ foo
$ echo ${bar}
42
```

<div class="fragment">
<pre><code data-trim>
foo() {
  local bar=42 # local
}
$ foo
$ echo ${bar} # no output
</code></pre>
</div>



## Serving static files
`HTTP/1.1 GET /index.html`


### Content Type

`text/html`, `application/json`, etc.


### Associative Arrays
(AKA Map, Dictionary, Hash)
```bash
declare -A content_types=([js]=text/javascript
                          [html]=text/html)
```


### Find content type for extension
```bash
$ echo ${content_types[html]}
text/html
```

<div class="fragment">
<p>With default:</p>
<pre class="bash"><code data-trim>
$ echo ${content_types[foo]-text/plain}
text/plain
</code></pre>
</div>


### Get file extension
```bash
$ file=public/index.html
$ echo ${file##*.}
html
```


### Serving a file

```bash
local file="public/${path}"
if [ -f "${file}" ]; do
  local ext="${file##*.}"
  local type="${content_types[${ext}]}"
  printf "HTTP/1.1 200 OK\r\n"
  printf "Content-Type: ${type}\r\n\r\n"
  cat "${file}"
fi
```



## Actor Model
<ol>
<li class="fragment">Asynchronous process</li>
<li class="fragment">Queue for communication</li>
</ol>


But this is Bash...


![Tom Cruise](tomcruise.jpg)


### John Agar
![John Agar](johnagar.jpg)

B-List Actor


![The Mole People poster](molepeople.jpg)


![Revenge of the Creature poster](creature.jpg)


### B-Movie Actor Model
<ol>
<li>Asynchronous process<span class="fragment">: a subshell</span></li>
<li>Queue for communication<span class="fragment">: a named pipe (FIFO)</span></li>
</ol>


### Create an Actor
```bash
agar() {
  # ...
}
```


### Get actor name and args
```bash
local name=$1
local args=${@:1}
```


### Create Pipe
```bash
mkdir -p fifos
local fifoname=fifos/${name}
mkfifo ${fifoname}
```


### Spawn Actor
```bash
{ 
    queue=${fifoname} ${name} ${args[@]}
    rm -f ${fifoname}
} &
```


### "Send" Helper
```bash
send() {
    local dest=$1
    local message=${*:1}
    echo "${message}" > fifos/${dest} &
}
```


### Create an Actor
```bash
agar main
```


### "Main" Function
```bash
main() {
  while true; do
    serve_with_coproc &
    read < ${queue}
  done
}
```


### Signaling Main
```bash
# ...
read req_line
send main "continue"
# ...
```


### Take that, Erlang!



## Persistence


### JSON Messages
```json
{
  "name": "Avdi",
  "text": "Testing 1 2 3"
}
```


### Database
Everyone's favorite NoSQL document database...


### PostgreSQL!
<img src="postgres.png" style="border: none; box-shadow: none; background: transparent"/>


### Schema
```sql
CREATE TABLE messages (
    id        serial PRIMARY KEY,
    content   json,
    posted_at timestamptz DEFAULT now()
);
```


```sql
INSERT INTO messages (content) VALUES
('{
    "name": "Bullwinkle",
    "text": "Hey Rocky, watch me pull a rabbit out of my hat"
  }');
INSERT INTO messages (content) VALUES
('{ "name": "Rocky", "text": "Again?" }');
INSERT INTO messages (content) VALUES
('{ "name": "Bullwinkle", "text": "Presto!" }');
```


```sql
SELECT content->>'text' AS text
FROM messages
WHERE content->>'name' = 'Bullwinkle';
```
```txt
                      text                     
-------------------------------------------------
 Hey Rocky, watch me pull a rabbit out of my hat
 Presto!
(2 rows)
```


No need to parse JSON in Bash!



## The Myth
"Bash isn't Webscale"


I got it working... barely.


### Analysis
- Slow
- Unreliable
- Only works in Firefox (?!)
- Leaks Processes
- FIFOs are a pain to work with
- So is netcat


This was a *terrible* idea!

![Car with a log for a wheel](manhole1.png)


### Myth: Confirmed

<img alt="Myth Confirmed" src="confirmed.png" style="border: none; box-shadow: none; background: transparent"/>


## Dumb ideas


### Gherkin

#### A Lisp in Bash

> "The one dependency we can count on"

https://github.com/alandipert/gherkin


### shasm

#### An x86 assembler in Bash

> "It just continuously cracks me up."

&mdash;Rick Hohensee


> "What a pointless waste of time"

&mdash;My brain


### A Kernel in Rust

> "If you ask questions that are really dumb, eventually you know 
> things"

&mdash; Julia Evans

http://jvns.ca/


"How do I make the next Instagram?"


"How do I write a kernel?"


### Outlier Knowledge

Comes from asking outlier questions


### Experimentation, not Demonstration
![Jamie Hyneman again](conclusion.png)


![Kids with mirrors](boat.png)


> When we experiment&mdash;when we try things, and we fail&mdash;we start to ask why, and that’s when we learn.

&ndash;Jamie Hyneman


A story to tell


Abstractions stripped away

- SQL
- HTTP


Greater familiarity with tools

- netcat
- psql
- stdbuf
- mitmproxy


Increased comfort with Bash scripting

- Functions
- Associative arrays
- Coprocesses


It was **fun**


### Experiment!

![JATO Rocket Car](jatocar.png)


## Thank You!

Code: https://github.com/avdi/walrus

I make screencasts: **RubyTapas.com**

@avdi / avdi@avdi.org


- MythBusters: &copy; 2014 Discovery Communications, LLC.
- Photo of Tom Cruise by Gareth Cattermole &copy; 2010 Getty Images
- "The Mole People" poster &copy; Universal Pictures
- All others either unknown or in the public domain
