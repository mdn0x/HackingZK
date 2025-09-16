# Introduction to Bluetooth

`Bluetooth`, a wireless technology standard, is designed for transferring data over short distances from fixed and mobile devices. The technology operates by establishing `personal area networks` (PANs) using `radio frequencies` in the `ISM band from 2.402 GHz to 2.480 GHz`. Conceived as a `wireless alternative to RS-232 data cables`, Bluetooth has been widely adopted due to its flexibility and ease of use. The name `Bluetooth` is derived from the epithet of a tenth-century king, `Harald Bluetooth`, who unified Denmark and Norway. The technology's use span numerous devices, such as smartphones, laptops, audio devices, and many IoT devices, with annual device shipments expected to `exceed 7 billion by 2026`, according to the [Bluetooth 2022 Market Update](https://www.bluetooth.com/2022-market-update).

Bluetooth functionality is based on several key concepts, including `device pairing`, `piconets`, and data `transfer protocols`. The first step in establishing a Bluetooth connection is the `pairing process`. This involves two devices `discovering` each other and establishing a connection:

1. `Discovery`: One device makes itself `discoverable`, broadcasting its presence to other Bluetooth devices within range.
2. `Pairing Request`: A second device finds the discoverable device and sends a `pairing request`.
3. `Authentication`: The devices authenticate each other through a process involving a shared secret, known as a `link key` or `long-term key`. This may involve entering a PIN on one or both devices.

Once the devices are paired, they remember each other's details and can automatically connect in the future without needing to go through the pairing process again.

After pairing, Bluetooth devices form a network known as a `piconet`. This collection of devices connected via Bluetooth technology consists of one `main` device and up to seven `active client` devices. The `main` device coordinates communication within the piconet.

Multiple piconets can interact to form a larger network known as a `scatternet`. In this configuration, some devices serve as bridges, participating in multiple piconets and thus enabling inter-piconet communication.

Bluetooth connections facilitate both `data` and `audio communication`, with data transfer occurring via packets. The primary device in the piconet dictates the schedule for packet transmission. The Bluetooth specification identifies two types of links for data transfer:

1. `Synchronous Connection-Oriented (SCO) links`: Primarily used for audio communication, these links reserve slots at regular intervals for data transmission, guaranteeing steady, uninterrupted communication ideal for audio data.
2. `Asynchronous Connection-Less (ACL) links`: These links cater to transmitting all other types of data. Unlike SCO links, ACL links do not reserve slots but transmit data whenever bandwidth allows.

As the utilisation of Bluetooth expands, so does the need for a comprehensive understanding of the associated security risks. The convenience of Bluetooth technology has inadvertently opened a new avenue for potential attacks that could compromise personal and organisational security.

## Risks of Bluetooth

In Bluetooth technology, risk can be defined as any potential event that exploits the Bluetooth connection to `compromise` data or systems' `confidentiality`, `integrity`, or `availability`. Bluetooth risks often emanate from the wireless nature of the technology that allows for `remote, unauthorised access` to a device, thereby presenting opportunities for nefarious activities such as eavesdropping, data theft, or malicious control of a device.

The array of risks associated with Bluetooth can be broadly classified into several categories:

1. `Unauthorised Access`: This risk involves unauthorised entities gaining unsolicited access to Bluetooth-enabled devices. Attackers can exploit vulnerabilities to take control of the device or eavesdrop on data exchanges, potentially compromising sensitive information and user privacy.
2. `Data Theft`: Bluetooth-enabled devices store and transmit vast amounts of personal and sensitive data. The risk of data theft arises when attackers exploit vulnerabilities to extract this data without authorisation. Stolen information may include contact lists, messages, passwords, financial details, or other confidential data.
3. `Interference`: Bluetooth operates on the 2.4 GHz band, which is shared by numerous other devices and technologies. This creates a risk of interference, where malicious actors may disrupt or corrupt Bluetooth communication. Intentional interference can lead to data loss, connection instability, or other disruptions in device functionality.
4. `Denial of Service (DoS)`: Attackers can launch Denial of Service attacks on Bluetooth-enabled devices by overwhelming them with an excessive volume of requests or by exploiting vulnerabilities in Bluetooth protocols. This can result in the targeted device becoming unresponsive, rendering it unable to perform its intended functions.
5. `Device Tracking`: Bluetooth technology relies on radio signals to establish connections between devices. Attackers can exploit this characteristic to track the physical location of Bluetooth-enabled devices. Such tracking compromises the privacy and security of device owners, potentially leading to stalking or other malicious activities.

## Bluetooth Attacks

Over the years, several classifications of Bluetooth attacks have been identified, each with its unique characteristics and potential risks. Understanding these attack types is essential for promoting awareness and implementing effective security measures. The following table provides a comprehensive overview of various Bluetooth attacks and their descriptions:

|Bluetooth Attack|Description|
|---|---|
|Bluejacking|Bluejacking is a relatively harmless type of Bluetooth hacking where an attacker `sends unsolicited messages` or business cards to a device. Although it doesn't pose a direct threat, it can infringe on privacy and be a nuisance to the device owner.|
|Bluesnarfing|Bluesnarfing entails `unauthorised access to a Bluetooth-enabled device's data`, such as contacts, messages, or calendar entries. Attackers exploit vulnerabilities to retrieve sensitive information without the device owner's knowledge or consent. This could lead to privacy violations and potential misuse of personal data.|
|Bluebugging|Bluebugging enables an attacker to `control a Bluetooth device`, including making calls, sending messages, and accessing data. By exploiting security weaknesses, attackers can gain unauthorised access and manipulate the device's functionalities, posing a significant risk to user privacy and device security.|
|Car Whisperer|Car Whisperer is a Bluetooth hack that `specifically targets vehicles`. Attackers exploit Bluetooth vulnerabilities to remotely unlock car doors or even start the engine without physical access. This poses a serious security threat as it can lead to car theft and compromise the safety of vehicle owners.|
|Bluesmacking & Denial of Service|Denial of Service (DoS) attacks leverage vulnerabilities in Bluetooth protocols to `disrupt or disable the connection between devices`. Bluesmacking, a specific type of DoS attack, involves sending excessive Bluetooth connection requests, overwhelming the target device and rendering it unusable. These attacks can disrupt normal device operations and cause inconvenience to users.|
|Man-in-the-Middle|Man-in-the-Middle (MitM) attacks intercept and manipulate data exchanged between Bluetooth devices. By `positioning themselves between the communicating devices`, attackers can eavesdrop on sensitive information or alter the transmitted data. MitM attacks compromise the confidentiality and integrity of Bluetooth communications, posing a significant risk to data security.|
|BlueBorne|Discovered in 2017, BlueBorne is a critical Bluetooth vulnerability that allows an attacker to take `control of a device without requiring any user interaction or device pairing`. Exploiting multiple zero-day vulnerabilities, BlueBorne presents a severe threat to device security and user privacy. Its pervasive nature makes it a significant concern across numerous Bluetooth-enabled devices.|
|Key Extraction|Key extraction attacks aim to `retrieve encryption keys` used in Bluetooth connections. By obtaining these keys, attackers can decrypt and access sensitive data transmitted between devices. Key extraction attacks undermine the confidentiality of Bluetooth communications and can result in exposure of sensitive information.|
|Eavesdropping|Eavesdropping attacks involve `intercepting and listening to Bluetooth communications`. Attackers capture and analyse data transmitted between devices, potentially gaining sensitive information such as passwords, financial details, or personal conversations. Eavesdropping attacks compromise the confidentiality of Bluetooth communications and can lead to severe privacy violations.|
|Bluetooth Impersonation Attack|In this attack type, an attacker `impersonates a trusted Bluetooth device` to gain unauthorised access or deceive the user. By exploiting security vulnerabilities, attackers can trick users into connecting to a malicious device, resulting in data theft, unauthorised access, or other malicious activities. Bluetooth impersonation attacks undermine trust and integrity in Bluetooth connections, posing a significant risk to device security and user trust.|
# Bluetooth Legacy Attacks
## Legacy Attacks

Considering the age of Bluetooth, many of the better-known and frequently cited Bluetooth attacks and vulnerabilities were discovered in the early to mid-2000s.

### Bluejacking

`Bluejacking` refers to a legacy attack involving `sending unsolicited messages` to Bluetooth-enabled devices. Unlike Bluesnarfing, Bluejacking does `not involve accessing` or stealing data from the targeted device. Instead, it uses the Bluetooth "business card" feature to `send anonymous messages`, typically in text or images. As such, it is often seen as a prank or nuisance rather than a serious security threat.

In a typical bluejacking scenario, the attacker starts by scanning for other nearby Bluetooth devices. The attacker then chooses a target from the discovered devices and proceeds to craft a text message or selects an image. Using the `Object Push Profile` (OPP), they send unsolicited messages or images to the targeted device.

The `Object Push Profile` (OPP) is a standard Bluetooth profile that facilitates the basic exchange of objects or files between devices. These objects can range from `Virtual Business Cards` (`vCards`), `calendar entries` (`vCalendars`), notes, or other forms of data encapsulated in a file.

Notably, this entire process can be accomplished without establishing a paired connection with the target device, making bluejacking a stealthy operation that leaves little trace other than an unexpected message on the recipient's device.

The impact of Bluejacking is generally limited to `annoyance` or `inconvenience`; however, in some instances, Bluejacking can be used maliciously. For example, it could be used as part of a `social engineering attack`, where the unsolicited message is designed to trick the recipient into revealing sensitive information or performing an action that compromises their security, such as installing malware onto their device.

Over time, the awareness about bluejacking spread, and manufacturers and developers began implementing security measures to address the vulnerability, such as changing default settings. These improvements made it significantly more difficult for unauthorised individuals to perform bluejacking.

Apple devices were susceptible to an attack very similar to Bluejacking via `AirDrop`. AirDrop is a proprietary service Apple provides that allows users to transfer files among supported Macintosh computers and iOS devices over Wi-Fi and Bluetooth—without using mail or a mass storage device.

AirDrop utilises Bluetooth to create a peer-to-peer network between devices. To share a file via AirDrop, the user selects the `Share` button on the document, photo, or webpage they wish to send, chooses AirDrop, and selects the device within the range they want to share with. When a file is received via AirDrop, it `appears as a notification`, allowing the user to either accept or decline. If the incoming AirDrop item is accepted, it is automatically saved into the user's Photos, Downloads, or Files folder on their device.

However, this feature has been exploited for spamming purposes.

If your AirDrop was set up to receive files from `Everyone`, this means that anyone within AirDrop range—roughly 30 feet (9 meters)—could share files with you.

In crowded areas such as concerts, subway trains, or aeroplanes, malicious users could send unsolicited photos or other files to any device set to receive from `Everyone`. The sending of unsolicited files is often termed `AirDrop spamming` or `cyber flashing`. However, Apple modified these settings in `iOS 16.2`, where the `Everyone` option had to be explicitly enabled before it automatically disabled itself after 10 minutes, thus only allowing your contacts to send you unsolicited files.

### Bluesnarfing

`Bluesnarfing` is a legacy attack that provides `unauthorised access` to a Bluetooth-enabled device to `extract information`, whose origins can be traced back to around 2003. The term derives from '`snarfing`,' a colloquial term in computer security, which denotes `unauthorised data acquisition`, and the 'blue' prefix refers to Bluetooth as the method of access. Thus, Bluesnarfing is essentially a data theft method exploiting Bluetooth connectivity.

Bluesnarfing entails several steps. The process begins with the attacker identifying an `active`, `vulnerable` Bluetooth device `within range`. Subsequently, the attacker `exploits a vulnerability` in the device's Bluetooth implementation, allowing `unauthorised access`. Once access is obtained, the attacker can `extract information` from the device, such as contact information, calendars, emails, text messages, and even media files. Notably, the attack can be perpetrated without alerting the device's owner, making it a surreptitious exploit.

The potential impact of Bluesnarfing is significant, given the nature of data that can be extracted. Personal information, including contact details and media files such as photos and videos, could potentially be used for identity theft or blackmail. Additionally, business-related information, such as meeting details or confidential emails, could be used for corporate espionage or blackmail.

The issue of Bluesnarfing prompted manufacturers and developers to take action. The [Bluetooth Special Interest Group (Bluetooth SIG)](https://en.wikipedia.org/wiki/Bluetooth_Special_Interest_Group), the organisation responsible for developing and standardising Bluetooth technology, released security advisories and guidelines to address the vulnerability. Device manufacturers released firmware updates and patches to fix security flaws and enhance the overall security of Bluetooth implementations. As a result of these efforts, Bluesnarfing has become less prevalent and is considered less of an issue for modern devices.

### Bluebugging

`Bluebugging` is a form of Bluetooth attack where an attacker gains `full control` over a Bluetooth-enabled device, allowing them to access and modify information, use the device to make calls, send text messages, and even connect to the internet. This is a much more severe and invasive threat than Bluesnarfing and Bluejacking, given the level of control it provides to the attacker.

Executing a Bluebugging attack is similar to Bluesnarfing, beginning with identifying an active, vulnerable Bluetooth device within range. The attacker then exploits vulnerabilities in the Bluetooth implementation of the device, often by tricking the user into believing they are pairing with a trusted device or `brute forcing` a Bluetooth `pairing PIN` on an implementation that does not prompt the user for confirmation. Once access is gained, the attacker can essentially `control the device` as if it were their own.

The impact of Bluebugging can be severe, given the `comprehensive control` it provides to the attacker. Personal information such as contact details, messages, and emails can be accessed and modified, or the device can be used to make calls or send messages. Additionally, the device’s microphone and camera can be remotely operated, turning the device into a `covert listening device`.

Maintaining device security, keeping devices updated with the latest firmware and security patches, and following recommended security practices to mitigate potential risks associated with Bluebugging and other Bluetooth-related vulnerabilities remains crucial.

### BlueSmacking

`Bluesmacking` is a `denial-of-service` (DoS) attack targeting Bluetooth-enabled devices. The attack exploits a vulnerability in the `L2CAP` Bluetooth protocol to `transfer large packets`, taking advantage of the limited processing capabilities of certain devices. Bluesmacking attacks were primarily executed using specialised software tools that generated and transmitted many Bluetooth packets.

Modern Bluetooth devices have implemented improved firmware and software solutions that enable better handling and filtering of Bluetooth packets. These enhancements have significantly reduced the effectiveness of BlueSmacking attacks and made it less of an issue for modern devices.
# Modern Bluetooth Attacks and Mitigation

## Modern Bluetooth Attacks

As we delve deeper into the realm of Bluetooth hacking, it's essential to understand that the landscape has evolved significantly since the days of legacy attacks such as `Bluesnarfing` or `Bluejacking`. While these older attacks have offered valuable lessons in cybersecurity, hackers are constantly devising newer and more sophisticated techniques to exploit Bluetooth vulnerabilities.

This page shifts our focus from legacy to modern Bluetooth hacking methods. We'll examine several modern attacks, including `BlueBorne`, `Key Negotiation of Bluetooth (KNOB) Attack`, and `Bluetooth Impersonation AttackS (BIAS)`.

### BlueBorne

`BlueBorne`, an attack discovered in 2017, emerged as a substantial threat in the field of cybersecurity, presenting hackers with the ability to `exploit Bluetooth connections and gain complete control over targeted devices`. The wide-ranging nature of the BlueBorne attack vector poses a significant risk to the vast array of devices equipped with Bluetooth capabilities. From conventional computers and mobile devices extending to IoT devices such as televisions, watches, cars, and medical appliances. What sets BlueBorne apart is its capability to `compromise devices without requiring them to be paired or set on discoverable mode`. The research team at `Armis Labs`, who unearthed this vulnerability, identified `eight zero-day vulnerabilities associated with BlueBorne`, revealing the existence of this attack vector and its potential impact. However, numerous other vulnerabilities are believed to remain undiscovered in various Bluetooth-enabled platforms.

These vulnerabilities have been extensively researched and confirmed to be operational, highlighting the significant risk BlueBorne poses. Exploiting these vulnerabilities, hackers can execute remote code on targeted devices and even carry out Man-in-The-Middle attacks. The versatility of the BlueBorne attack vector allows for a wide range of offences, making it a potent threat to device security and user privacy.

Blueborne stands as a stark reminder of the vulnerabilities present in Bluetooth technology. Its discovery led to increased awareness and emphasised the need for robust security measures in Bluetooth-enabled devices.

### KNOB

The `Key Negotiation of Bluetooth (KNOB)` attack represents a sophisticated and potentially devastating form of Bluetooth hacking. It exploits a flaw in the Bluetooth standard to `undermine the encryption` of Bluetooth connections. This vulnerability was discovered by three researchers: Daniele Antonioli from the Singapore University of Technology and Design, Nils Ole Tippenhauer from CISPA Helmholtz Center for Information Security, and Kasper Rasmussen from the University of Oxford. They published their findings, [Key Negotiation of Bluetooth (KNOB) Attack](https://knobattack.com/), in 2019.

Bluetooth connections rely on an encrypted link using a shared encryption key. During the `setup` of this encrypted link, both `devices agree` on the `encryption key's length`. The vulnerability that the KNOB attack exploits lies in this negotiation process.

The attacker intercepts the Bluetooth pairing communication and `forcibly sets the length` of the encryption key to its `minimum allowed size`, which is only `one byte`. With such a weak encryption key, it becomes trivial for the attacker to crack it through `brute force` methods, thereby gaining access to the encrypted communication.

One crucial aspect of the KNOB attack is that it `does not require knowledge of any previously shared link key`, making it an entirely new attack methodology compared to previous attacks on Bluetooth encryption.

Essentially `all devices` that adhere to the `Bluetooth standard`, including smartphones, tablets, laptops, and IoT devices, could be vulnerable to a KNOB attack. The inherent issue lies within the Bluetooth standard itself rather than specific implementations, making it a broad-spectrum vulnerability.

Following its discovery, patches were developed and rolled out to fix this issue in many devices. The best way to protect against a KNOB attack is by ensuring all your Bluetooth-capable devices are `updated` with the `latest software` and `firmware`. Moreover, the Bluetooth Special Interest Group (SIG), which oversees the Bluetooth standard's development, has updated the standard to require a minimum encryption key length of seven bytes going forward.

### BIAS

`Bluetooth Impersonation AttackS (BIAS)` is a novel attack discovered in 2020 discovered, again by Daniele Antonioli, Nils Ole Tippenhauer, and Kasper Rasmussen, and published in [BIAS: Bluetooth Impersonation AttackS](https://ieeexplore.ieee.org/document/9152758).

The BIAS attack targets the secure, simple pairing and connection processes employed by devices implementing the Bluetooth BR/EDR (Basic Rate/Enhanced Data Rate) specification or [Bluetooth Classic](https://www.bluetooth.com/learn-about-bluetooth/tech-overview/). This attack method enables the attacker to `authenticate themselves` by connecting with a victim's device and `masquerading` as a `device already paired` with the victim's.

The core vulnerability lies in the fact that during this impersonation process, the Bluetooth protocol `does not require mutual authentication` - it only requires that the `device initiating the connection verifies the other party`. This means that a malicious actor can impersonate a Bluetooth device `without knowing` the `long-term key` shared by the impersonated device.

Once this impersonation is successful, attackers can access critical functions such as reading and writing data on the victim's device or even establishing a full-fledged Man-in-The-Middle (MITM) attack.

BIAS attacks are potentially effective against `any device using the Bluetooth BR/EDR specification`, including smartphones, tablets, laptops, and certain IoT devices. Since this attack method exploits fundamental protocol weaknesses, it is agnostic to the device type or manufacturer.

BIAS attacks can be mitigated through `firmware updates` that address the specific vulnerabilities in the Bluetooth Classic protocol. Therefore, ensuring that your devices are up-to-date with their `latest firmware` and `software updates` is crucial in defending against BIAS attacks.

## Mitigation

Following best practices and implementing security measures is crucial to mitigate the risks of Bluetooth technology. The following recommendations can help users enhance the security of their Bluetooth-enabled devices:

1. `Keep Devices Updated`: Ensure you update the firmware and software of your Bluetooth-enabled devices regularly. Manufacturers frequently release updates and patches to address security vulnerabilities and enhance device security.
2. `Disable Bluetooth When Not Needed`: If you're not actively using Bluetooth, `switch it off` or disable it on your devices. You minimise potential attack opportunities by keeping Bluetooth off when it's not in use. This simple action can significantly lower the risk of unauthorised access and potential exploitation of Bluetooth vulnerabilities.
3. `Enable Device Pairing Authorisation`: Modern devices typically have this setting enabled by default, but older devices might default to `automatic pairing without authorisation`. Activate device pairing authorisation on your Bluetooth-enabled devices. This feature necessitates devices to be authenticated and authorised before forming a connection. Enabling this setting helps prevent unauthorised devices from accessing your Bluetooth-enabled devices, thereby protecting them from potential attacks.
4. `Limit Device Visibility`: Set your device to be invisible or undiscoverable when it's not being used to restrict its visibility to potential attackers.
5. `Exercise Caution in Public Settings`: Exercise caution when using Bluetooth in public spaces. `Do not pair with or accept connection requests from unknown devices`. Attackers might exploit Bluetooth vulnerabilities in crowded places or public networks to gain unauthorised device access.