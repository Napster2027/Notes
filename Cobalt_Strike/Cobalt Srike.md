## # `Introduction` -

Cobalt Strike is a commercial penetration testing tool, which gives security testers access to a large variety of attack capabilities. Cobalt Strike is threat emulation software. This is how its marketed, but in a simple form it’s a C2 framework. A C2 framework is a command-and-control solution for post exploitation, meaning the tool is used mostly to get a reverse shell on a Windows host which provides a variety of commands built-in that assist the attacker in completing objectives such as downloading files or escalating privileges. Cobalt Strike can be compared to Metasploit Meterpreter in some ways that it operates.

## # `Important Components` -

You may have heard the names Cobalt Strike, BEACON, and even team server used interchangeably, but there are some important distinctions between all of them.

1. ==Cobalt Strike== is the command and control (C2) application itself. This has two primary components: the team server and the client. These are both contained in the same Java executable (JAR file) and the only difference is what arguments an operator uses to execute it.

2. ==Team server== is the C2 server portion of Cobalt Strike. It can accept client connections, BEACON callbacks, and general web requests.
   • By default, it accepts client connections on TCP port 50050.
   • Team server only supports being run on Linux systems. 

3. ==Client== is how operators connect to a team server.
   • Clients can run on the same system as a Team server or connect remotely.
   • Client can be run on Windows, macOS or Linux systems. 

4. ==BEACON== is the name for Cobalt Strike's <mark style="background: #3D7EFFA6;">default malware payload used to create a connection to the team server</mark>.Active callback sessions from a target are also called "beacons". (This is where the malware family got its name.)There are two types of BEACON:
   
   • The ==Stager== is an optional BEACON payload. Operators can "stage" their malware by sending an initial small BEACON shellcode payload that only does some basic checks and then queries the configured C2 for the fully featured backdoor. <mark style="background: #3D7EFFA6;">Stagers are less common now due to to the high detections</mark> of breaking the payloads into separate parts.
   
   • The ==Full backdoor== can either be executed through a BEACON stager, by a "loader" malware family, or by directly executing the default DLL export "ReflectiveLoader". This backdoor runs in memory and can establish a connection to the team server through several methods.

5. ==Loaders== are not BEACON. BEACON is the backdoor itself and is typically executed with some other loader, whether it is the staged or full backdoor. Cobalt Strike does come with default loaders, but operators can also create their own using PowerShell, .NET, C++, GoLang, or really anything capable of running shellcode.

6. ==Listeners== are the Cobalt Strike component that <mark style="background: #3D7EFFA6;">payloads, such as BEACON, use to connect to a team server</mark>. Cobalt Strike supports several protocols and supports a wide range of modifications within each listener type. Some changes to a listener require a "listener restart" and generating a new payload. Some changes require a full team server restart.
   
   ==HTTP/HTTPS== is by far the most common listener type.

   • While Cobalt Strike includes a default TLS certificate, this is well known to defenders and blocked by many enterprise products ("signatured"). Usually operators will generate valid certificates, such as with LetsEncrypt, for their C2 domains to blend in.

   • Thanks to Malleable Profiles, operators can heavily configure how the BEACON network traffic will look and can masquerade as legitimate HTTP connections.

   • Operators can provide a list of domains/IPs when configuring a listener, and the team server will accept BEACON connections from all of them. Operators can also specify Host header values.
   
   ==DNS== listeners establish sessions to their team server using DNS requests for domains the team server is authoritative for. DNS listeners support two modes: 
   
   • ==Hybrid (DNS+HTTP)== is the default and uses DNS for a beacon channel and HTTP for a data channel.

   • ==Pure DNS== can also be enabled to use DNS for both beacon and data channels. This leverages regular A record requests to avoid using HTTPS and provide a stealthier, though slower method of communication.
   
   ==SMB== is a bind style listener and is most often used for chaining beacons. Bind listeners open a local port on a targeted system and wait for an incoming connection from an operator.
   
   ==Raw TCP== is a (newer) bind style listener and can also be used for chaining beacons.
   
   The final two listeners are less common, but they provide compatibility with other payload types.
   
   ==Foreign listeners== allow connections from Metasploit's Meterpreter backdoor to simplify passing sessions between the Metasploit framework and the Cobalt Strike framework.
   
   ==External C2== listeners provide a specification that operators can use to connect to a team server with a reverse TCP listener. Reverse listeners connect back and establish an external connection to an operator, instead of waiting for an incoming connection such as with "bind" listeners.

7. ==Redirectors== , Instead of having beacons connect directly to a team server, operators will sometimes use a redirector (or several) that accepts connections and forwards them to the team server. This has several advantages for operators, including being able to:
   • Cycle through multiple domains for a single BEACON connection
   • Replace detected/blocked redirectors without having to replace the underlying team server
   • Use high(er) reputation domains that help BEACON traffic blend in and avoid detection
   
   Operators can also use redirectors to filter out "suspicious" traffic, like scanners or hunting tools, to protect their team server, however there are typically still easy wins to track down team servers and redirectors.
   

8. ==Execute-Assembly== is a BEACON command that <mark style="background: #3D7EFFA6;">allows operators to run a .NET executable in memory on a targeted host</mark>. BEACON runs these executables by spawning a temporary process and injecting the assembly into it. In contrast to Aggressor Scripts, execute-assembly does allow operators to extend BEACON functionality. Assemblies run in this way will still be <mark style="background: #3D7EFFA6;">scanned by Microsoft's AMSI</mark> if it is enabled.

9. ==Malleable Profile== allows operators to extensively <mark style="background: #3D7EFFA6;">modify how their Cobalt Strike installation works</mark>. It is the most common way operators customize Cobalt Strike and has thus been heavily documented.
   
	 • Changes to a Malleable Profile require a team server restart and, depending on the change, may require regenerating payloads and re-spawning beacon sessions.
   
	• There are several robust open-source projects that generate randomized profiles which can make detection challenging. Still, operators will often reuse profiles (or only slightly modify them) allowing for easier detection and potentially attribution clustering.
   
	 • When analyzing samples, check GitHub and other public sources to see if the profile is open source
   
   ==Malleable Profiles== allow operators to customize a wide range of settings when they first launch their team server. The snippet that follows from a public profile is an example of how an operator could make BEACON traffic look like it's related to Amazon. The portions in blue (the set uri line and the client block), define how a BEACON payload behaves. Some of these values can be extracted from a BEACON sample.

- Malleable C2 Profile Auxilliary Section -

```c
set sample_name "Zsec Example Profile";
set host_stage "false"; 
set jitter "0";
set pipename "msagent_###"; # Each # is replaced witha random hex value.
set pipename_stager "status_##";
set sleeptime "60000"; # default sleep in ms, 1 minute
set ssh_banner "Cobalt Strike 4.4";
set ssh_pipename "postex_ssh_####";
set data_jitter "100"; 
set useragent "Not Cobalt Strike";
```  

	sample_name – used for profile management
	host_stage – ‘false’ for stageless and ‘true’ for staged
	jitter – setting jitter as a percentage on the sleep time of a beacon
	pipename – default name for named pipes
	sleeptime – default is 60000 (1 minute)
	ssh_banner – banner that shows for ssh beacons
	ssh_pipename – pipe name for ssh beacons
	data_jitter – enables the operator to append data
	useragent – sets User-Agent string


- Malleable C2 Profile HTTP Config Section -
  
  In addition to the auxiliary information at the top of the profile, the http-config section specifies additional aux information related to specifics applicable to all aspects of the profile. Such as headers to be sent in requests, whether X-Forwarded-For is to be trusted or not and if specific user agents are to be blocked or allowed. The httpconfig block has influence over all HTTP responses served by Cobalt Strike's web server.
  
```c
  http-config {
    set headers "Date, Server, Content-Length, Keep-Alive, Connection, Content-Type";
    header "Server" "Apache";
    header "Keep-Alive" "timeout=10, max=100";
    header "Connection" "Keep-Alive";
    set trust_x_forwarded_for "true";
    set block_useragents "curl*,lynx*,wget*";
    set allow_useragents "*Mozilla*";
}
```

	 	set headers – sets http headers between beacon and CS server
	 	trust_x_forwarded_for – use if your CS teamserver is behind a redirector (it should be)
	 	block_useragents – helpful for blocking specific user agents that you don’t want touching your server
	 	allow_useragents – whitelisting of specific user agents that can connect to the team server 


- Malleable C2 Profile TLS Certs Section -
  
  When using a HTTPS listener, CS gives the option for using signed HTTPS certificates for C2 communications. There are multiple options when setting this up ranging from none to signed by trusted authority.
  
  3 options:
  - none
  - self-signed
  - signed by trusted authority
  
  ```c
  https-certificate {
	# Option 1: Create a signed certificate with Java Keystore tool
	set keystore "/pathtokeystore";
    set password "password";
    
    # Option 2: Self Signed with vendor specifics
    set C   "US";
    set CN  "jquery.com";
    set O   "jQuery";
    set OU  "Certificate Authority";
    set validity "365";
    
}
```

Red Team engagements  use multiple redirectors that sit in front of CS team server that have their own valid cert and pass traffic through private connections such as SSH reverse tunnels or pass-through proxies in AWS/Azure. CS Team Server is never exposed to the internet, only whitelist the CS Team server port for global worldwide access through AWS or Azure depending on client needs. The idea of not having a CS server exposed at all is your best option.


- Malleable C2 Profile Client and Server Interactions Section -

**GET SECTION** :-

The most customizable aspect of the profile is being able to specify which sections act in which ways, the main ones are GET and POST specifying how traffic is intercepted and how data is chunked. An example GET and POST section are shown below complete with both client and server interactions.

```c
http-get "WindowsUpdates" {
  
  set verb "GET";
  set uri "/c/msdownload/update/others/2016/12/29136388_";
  
  client {
  
		header"Accept" "*/*";
	    header"Host" "download.windowsupdate.com";

		#session metadata 
		metadata {
			base64url;
			append ".cab";
			uri-append;
	    }
  }
  
  server {
	header "Content-Type" "application/vnd.ms-cab-compressed";
    header "Server" "Microsoft-IIS/8.5";
    header "MSRegion" "N. America";
    header "Connection" "keep-alive";
    header "X-Powered-By" "ASP.NET";

	#Beacon's tasks 
	output { 
		print; 
	} 
}
```

	set uri – the URI your beacon will call back to (hard-coded)
	client - specifies info sent by the beacon
	metadata – cookies can be set, C2 data can be hidden here
	server – details how the server responds to C2 traffic


The main sections of the profile are broken up into uri, client, server and the contents held within each. Breaking the above section down:

- ==set uri==: Specifies the URI that the beacon will call back to, this is chosen at random from the list at the time of generation of the beacon, initially one would assume these are round robin but unfortunately not. Each beacon variant will have one URI hard coded for both post and get, which is good news for defenders attempting to identify traffic in NetFlow data.

- ==The client section==: details the information sent and shown by the beacon on the target host, this dictates how traffic is chunked and sent and it also specifies how information is encoded, there are multiple options available for this. In addition, the profile enables you to set specific headers which is especially important if a specific site or endpoint is being emulated as this will show in the HTTP traffic. It also specifies what the expected host header is on traffic, this enables differentiating between false HTTP traffic and legitimate C2 traffic.

- ==The metadata section==: specifies where things such as cookies can be set, this is an additional place where data can be hidden on C2 communications, typically data is sent in either a specific header or a cookie value which can be specified and set to anything. When red teaming a client it is often common practice to profile users' browsers and expected traffic in an environment to enable better blending in. When CS's Beacon "phones home" it sends metadata about itself to the CS team server.

- ==The server section==: details how the server responds to C2 traffic, the example above tells the server to respond with raw data in its encrypted form however this can be customized in the same way as the client specifying key areas where things should be encoded

There are a few options available when it comes to data encoding and transformation. For example, you may choose to NetBIOS encode the data to transmit, prepend some information, and then base64 encode the whole package.

-   ==base64== - Base64 encode data that is encapsulated in various sections, in the enable above the cookie value cf_ contains encoded metadata to be sent back to the CS server.
- ==base64url== - URL-safe Base64 Encode, this is typically used when sending data back in a URL parameter and the data needs to be URL safe to not break the communication stream.
- ==mask== - XOR mask w/ random key, this encodes and encrypts the data within a XOR stream with a random key, typically used in combination with other encoding to obfuscate the data stream.
- ==netbios== - NetBIOS Encode 'a' it encodes as NetBIOS data in lower case.
- ==netbiosu== - NetBIOS Encode 'A', another form of NetBIOS encoding.

**POST SECTION** :-

```c
http-post "Windows Updates" {
  
  set verb "POST";
  set uri "/c/msdownload/update/others/2016/12/3215234_";
  
  client {

    header "Accept" "*/*";

	#session ID 
	id {
      prepend "download.windows update.com/c/";
      header "Host";
    }
	
	#Beacon's responses 
	output { 
		base64url; 
		append ".cab"; 
		uri-append; 
		} 
	} 
	
	server { 
		header "Content-Type" "application/vnd.ms-cab-compressed";
		header "Server" "Microsoft-IIS/8.5"; 
		header "MSRegion" "N. America"; 
		header "Connection" "keep-alive"; 
		header "X-Powered-By" "ASP.NET"; 
		
		#empty 
		output { 
			print; 
			} 
		}
}
```

Again, like the GET section above, the POST section states how information should be sent in a POST request, it has the added benefit that specifics such as body content and other parameters can be set to enable you to blend in.


- Malleable C2 Profile Post Exploitation Section -

Customizing the GET and POST requests is just the beginning, the next few sections of the profile is where the magic of post exploitation customization lives including how the beacon looks in memory, how migration and beacon object files affect the indicators of compromise and much more. These sections are so important when running post commands within the beacons or how your payloads are injected into memory.

```c
post-ex {
    
    set spawnto_x86 "%windir%\\syswow64\\dllhost.exe";
    set spawnto_x64 "%windir%\\sysnative\\dllhost.exe";
    set obfuscate "true";
    set smartinject "true";
    set amsi_disable "false";
    set keylogger "GetAsyncKeyState";
    set threadhint "module!function+0x##"
}
```

- ==spawnto_x86|spawnto_x64== - Specifies the process that will be hollowed out and new beacon process be created inside, this can typically be set to anything however it is recommended not to use the following "csrss.exe","logoff.exe","rdpinit.exe","bootim.exe","smss.exe","userinit.exe","sppsvc.exe". In addition, selecting a binary that does not launch with user account control is key(UAC). To add additional stealthy and blending techniques, you can add parameters to the spawnto command: set spawnto_x86 "%windir%\syswow64\dllhost.exe -k netsvcs";.

- ==obfuscate== - The obfuscate option scrambles the content of the post-exploitation DLLs and settles the post-ex capability into memory in a more operational security-safe manner.

- ==smartinject== - This directs Beacon to embed key function pointers, like GetProcAddress and LoadLibrary, into its same-architecture post-ex DLLs. This allows post-ex DLLs to bootstrap themselves in a new process without shellcode-like behavior that is detected and mitigated by watching memory accesses to the PEB and kernel32.dll.
  
- ==amsi_disable== - This option directs powerpick, execute-assembly, and psinject to patch the AmsiScanBuffer function before loading .NET or PowerShell code. This limits the Antimalware Scan Interface visibility into these capabilities. There are additional things that can be done post exploitation with the likes of beacon object files(BOFS).

- ==keylogger== - The GetAsyncKeyState option (default) uses the GetAsyncKeyState API to observe keystrokes. The SetWindowsHookEx option uses SetWindowsHookEx to observe keystrokes, this can be tuned even more within the TeamServer properties.

- ==Threadhint== - allows multi-threaded post-ex DLLs to spawn threads with a spoofed start address. Specify the thread
hint as "module!function+0x##" to specify the start address to spoof. The optional 0x## part is an offset added to the
start address.


- Malleable C2 Profile Process Injection Section -

```c
process-inject {
    set allocator "NtMapViewOfSection"; # or VirtualAllocEx
    set min_alloc "24576";
    set startrwx "false";
    set userwx   "false";

    transform-x86 {
        prepend "\x90\x90";
        #append "\x90\x90";
    }

    transform-x64 {
        prepend "\x90\x90";
        #append "\x90\x90";
    }
    
     execute {

        CreateThread "ntdll!RtlUserThreadStart";
        CreateThread;
        NtQueueApcThread-s;
        CreateRemoteThread;
        RtlCreateUserThread;
        SetThreadContext; 
    }
}
```

The various sections are defined as follows :

• ==set allocator== - Allows setting a remote memory allocation using one of two techniques: VirtualAllocEx or NtMapViewOfSection

• ==min_alloc== - Minimium memory allocation size when injecting content, very useful when it comes to being specific.

• ==set startrwx== - Use RWX as initial permissions for injected or BOF content. Setting this to false means that your memory segment will have RW permissions. When BOF memory is not in use the permissions will be set based on this setting.

• ==set userwx== – Setting this to false is asking the Beacon’s loader to avoid RWX permissions. Memory segments with these permissions will attract extra attention from analysts and security products.

• ==transform-x86 transform-x64== - Transform injected content to avoid signature detection of first few bytes. Only supports prepend and append of hex-based bytes.

The execute section controls the methods that the Beacon will use when it needs to inject code into a process. Beacon examines each option in the execute block, determines if the option is usable for the current context, tries the method when it is usable, and moves on to the next option if code execution did not happen.

- ==CreateThread== - current process only aka self-injection

- ==CreateRemoteThread== - Vanilla cross process injection technique. Doesn't cross session boundaries

- ==NtQueueApcThread|-s== - This is the "Early Bird"injection technique. Suspended processes (e.g., post-ex jobs) only.

- ==RtlCreateUserThread==- Risky on XP-era targets; uses RWX shellcode for x86->x64 injection.

- ==SetThreadContext== - Suspended processes (e.g. post-ex jobs only)

By default, a profile only contains one block of GET and POST however it is possible to pack variations of the current profile by specifying variant blocks.

```c
http-get "GET Azure" {
	client {
		parameter "api" "AZ_example";
        header "Cookie" "SomeValue";
	}
```


- Malleable C2 Profile Memory Indicators Section -

```c
stage {
    set userwx         	"false";
    set stomppe        	"true";
    set obfuscate      	"true";
    set name           	"srv.dll";
    set cleanup        	"true";

    # Values captured using peclone against a Windows 10 version of explorer.exe
    set checksum        "0";
    set compile_time    "11 Nov 2016 04:08:32";
    set entry_point     "650688";
	set image_size_x86  "4661248";
	set image_size_x64  "4661248";
	set rich_header     "\x3e\x98\xfe\x75\x7a\xf9\x90\x26\x7a\xf9\x90\x26\x7a\xf9\x90\x26\x73\x81\x03\x26\xfc\xf9\x90";

    # transform the x64 rDLL stage
    transform-x64 {
        strrep 			"This program cannot be run in DOS mode" "";
        strrep 			"beacon.dll" "";
        strrep 			"beacon.x64.dll" "";
        strrep 			"beacon.x32.dll" "";
    }
    
        # transform the x86 rDLL stage
    transform-x86 {
        strrep 			"ReflectiveLoader" "run";                    
        strrep 			"This program cannot be run in DOS mode" "";
        strrep 			"beacon.dll" "";
        strrep 			"beacon.x64.dll" "";
        strrep 			"beacon.x32.dll" "";
    }
```

	stomppe – ask ReflectiveLoader to stomp MZ, PE, and e_lfanew values after loading
	beacon payload
	name – exported name of the beacon DLL
	cleanup – ask beacon to free memory from the Reflective DLL that created it
	checksum –default is zero, checksum value in beacon’s PE header
	compile_time – sets the time that the PE was compiled
	entry_point – EntryPoint in the beacon’s PE header
	image_size_x86| image_size_x64 – SizeofImage value in beacon’s PE header
	rich_header – meta-information inserted by the compiler
	transform-x86|transform-x64 – transforms the beacon’s reflective DLL stage


10. ==Beacon Object Files (BOFs)== are a fairly recent Cobalt Strike feature that allows operators to extend BEACON post-exploitation functionality. <mark style="background: #3D7EFFA6;">BOFs are compiled C programs that are executed in memory on a targeted host</mark>. In contrast to Aggressor Scripts, <mark style="background: #3D7EFFA6;">BOFs are loaded within a BEACON session and can create new BEACON capabilities</mark>. Additionally, compared to other BEACON post-exploitation commands like execute-assembly, BOFs are relatively stealthy as they run within a BEACON session and <mark style="background: #3D7EFFA6;">do not require a process creation or injection</mark> .
   
   Beacon Object Files are <mark style="background: #3D7EFFA6;">single file C programs that are run within a BEACON session</mark>. BOFs are expected to be small and run for a short time. Since BEACON sessions are single threaded, <mark style="background: #3D7EFFA6;">BOFs will block any other BEACON commands while they are executing</mark>.

- single-file C programs that must include “beacon.h” in the same directory.

- limited to Windows APIs, internal beacon APIs and custom functions.

- no linking involved, not an exe
	• replace ‘main’ with ‘go’ for entry point
	• every single function must be imported

- one action and done, not for long-running jobs (use reflective DLL for those)

- executes inside your beacon, no fork n’ run here

- must use inline-execute

- normal C functions cannot be used, you’ll get a linking error

- Windows APIs must be declared, there is no Import Address Table

**What are the advantages of using BOF’s?**

One of the key roles of an command & control platform is to provide ways to use external post-exploitation functionality. Cobalt Strike already has tools to use PowerShell, .NET, and Reflective DLLs. These tools rely on an OPSEC expensive fork & run pattern that involves a process create and injection for each post-exploitation action. BOFs have a lighter footprint. They run inside of a Beacon process and are cleaned up after the capability is done. 

BOFs are also very small. A UAC bypass privilege escalation Reflective DLL implementation may weigh in at 100KB+. The same exploit, built as a BOF, is <3KB. This can make a big difference when using bandwidth constrained channels, such as DNS. 

Finally, BOFs are easy to develop. You just need a Win32 C compiler and a command line. Both MinGW and Microsoft's C compiler can produce BOF files. You don't have to fuss with project settings that are sometimes more effort than the code itself.


**How does it work?**

To Beacon, a BOF is just a block of position-independent code that receives pointers to some Beacon internal APIs.

To Cobalt Strike, a BOF is an object file produced by a C compiler. Cobalt Strike parses this file and acts as a linker and loader for its contents. This approach allows you to write position-independent code, for use in Beacon, without tedious gymnastics to manage strings and dynamically call Win32 APIs.


**What are the disadvantages of BOFs?**

BOFs are single-file C programs that call Win32 APIs and limited Beacon APIs. Don't expect to link in other functionality or build large projects with this mechanism.

Cobalt Strike does not link your BOF to a libc. This means you're limited to compiler intrinsics (e.g., \_\_stosb onVisual Studio for memset), the exposed Beacon internal APIs, Win32 APIs, and the functions that you write. Expect that a lot of common functions (e.g., strlen, stcmp, etc.) are not available to you via a BOF.

BOFs execute inside of your Beacon agent. If a BOF crashes, you or a friend you value will lose an access. Write your BOFs carefully.

Cobalt Strike expects that your BOFs are single-threaded programs that run for a short period of time. BOFs will block other Beacon tasks and functionality from executing. There is no BOF pattern for asynchronous or longrunning tasks. If you want to build a long-running capability, consider a Reflective DLL that runs inside of a sacrificial process.


11. ==Aggressor Scripts== are <mark style="background: #3D7EFFA6;">macros that operators can write and load in their client to streamline their workflow</mark>. These are loaded and executed within the client context and <mark style="background: #3D7EFFA6;">don't create new BEACON functionality</mark>, so much as automate existing commands. They are written in a Perl-based language called "Sleep" which Raphael Mudge (the creator of Cobalt Strike) wrote.

	- Aggressor scripts are only loaded into an operator's local Client. They are not loaded into other operators clients, the team server, or BEACON sessions (victim hosts). 


Aggressor Scripts can run BOF’s within the Cobalt Strike client. Most BOF’s released to the public include an aggressor script to help the user and client understand what to do and how to interact with the BOF


  - Aggressor script files have an extension of “.cna”.


This feature within CS is highly used during red team engagements.

**References** : 
- [Cobalt Strike Profiles](https://blog.zsec.uk/cobalt-strike-profiles/)
- [Getting started with Cobalt Strike Tutorial](https://hub.packtpub.com/red-team-tactics-getting-started-with-cobalt-strike-tutorial/) 

