
javax.net.ssl.SSLProtocolException: Read error: ssl=0xb400007263e78758: Failure in SSL library, usually a protocol error
error:10000416:SSL routines:OPENSSL_internal:SSLV3_ALERT_CERTIFICATE_UNKNOWN (external/boringssl/src/ssl/tls_record.cc:489 0xb400007273eb2670:0x00000003)
	at com.android.org.conscrypt.NativeCrypto.ENGINE_SSL_read_direct(Native Method)
	at com.android.org.conscrypt.NativeSsl.readDirectByteBuffer(NativeSsl.java:685)
	at com.android.org.conscrypt.ConscryptEngine.readPlaintextDataDirect(ConscryptEngine.java:997)
	at com.android.org.conscrypt.ConscryptEngine.readPlaintextData(ConscryptEngine.java:981)
	at com.android.org.conscrypt.ConscryptEngine.unwrap(ConscryptEngine.java:776)
	at com.android.org.conscrypt.ConscryptEngine.unwrap(ConscryptEngine.java:647)
	at com.android.org.conscrypt.ConscryptEngine.unwrap(ConscryptEngine.java:612)
	at com.android.org.conscrypt.ConscryptEngineSocket$SSLInputStream.processDataFromSocket(ConscryptEngineSocket.java:928)
	at com.android.org.conscrypt.ConscryptEngineSocket$SSLInputStream.readUntilDataAvailable(ConscryptEngineSocket.java:894)
	at com.android.org.conscrypt.ConscryptEngineSocket$SSLInputStream.read(ConscryptEngineSocket.java:867)
	at java.io.DataInputStream.readFully(DataInputStream.java:203)
	at rikka.shizuku.U0.b(SourceFile:33)
	at rikka.shizuku.U0.a(SourceFile:167)
	at rikka.shizuku.Qs.j(SourceFile:33)
	at rikka.shizuku.Q4.h(SourceFile:6)
	at rikka.shizuku.ba.run(SourceFile:102)
	at rikka.shizuku.A0.run(SourceFile:45)
	at rikka.shizuku.xp.run(SourceFile:3)
	at rikka.shizuku.Y7.run(SourceFile:89)
  # Introduction

Shizuku can help normal apps uses system APIs directly with adb/root privileges with a Java process started with app_process.

The name Shizuku comes from [a character](https://danbooru.donmai.us/posts/3553474).

## Why was Shizuku born?

The birth of Shizuku has two main purposes.

1. Provide a convenient way to use system APIs
2. Convenient for the development of some apps that only requires adb permissions

## Shizuku vs. "Old school" method

### "Old school" method

For example, to enable/disable components, some apps that require root privileges execute `pm disable` directly in `su`.

1. Execute `su`
2. Execute `pm disable`
3. (pre-Pie) Start the Java process with app_process ([see here](https://android.googlesource.com/platform/frameworks/base/+/oreo-release/cmds/pm/pm))
4. (Pie+) Execute the native program `cmd` ([see here](https://android.googlesource.com/platform/frameworks/native/+/pie-release/cmds/cmd/))
5. Process the parameters, interact with the system server through the binder, and process the result to output the text result.

Each of the "Execute" means a new process creation, su internally uses sockets to interact with the su daemon, and a lot of time and performance are consumed in such process. (Some poorly designed app will even execute `su` **every time** for each command)

The disadvantages of this type of method are:

1. **Extremely slow**
2. Need to process the text to get the result
3. Features are subject to available commands
4. Even if adb has sufficient permissions, the app requires root privileges to run

### Shizuku method

The Shizuku app will direct the user to run a process (Shizuku service process) using root or adb.

1. When the app process starts, the Shizuku service process sends the binder to the app process.
2. The app interacts with the Shizuku service through the binder, and the Shizuku service process interacts with the system server through the binder.

The advantages of Shizuku are:

1. Minimal extra time and performance consumption
2. It is almost identical to the direct invocation API experience (app developers only need to add a small amount of code)
