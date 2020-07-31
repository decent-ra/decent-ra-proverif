# DecentRaVerif
Verification of the Decent RA protocol, by using the ProVerif verification tool.

## Process Overview
The following figure shows that we modeled DecentRA framework with 6 different processes.
![Process Overview](README-src/ProcOverview.svg)

The next figure shows that the honest process is splited into an infinite replication of IAS processes, and an infinit replication of enclave platforms. Each enclave platform creates its own local report key, and further splits the process into infinit replication of Decent Servers and Decent Apps.
![Process Overview - More Details](README-src/ProcOverview-detailed.svg)

## Adversarial Model

* We assume
	* the correctness of all existing cryptographic algorithms (e.g. hash, DSA, DHKE)
	* the enclave platform is not compromised

* Attackers may not
	* access the private IAS report key, provisioning key, and local report key
	* access any private key held by a legitimate enclave
	* alter the behavior of the enclave

* Attackers may
	* know the behavior of the enclave
	* control all message channels
	* create their own Decent Apps or Decent Servers, and access their private keys
	* access the private keys held by revoked Decent Apps

## Attacker Processes

Since attackers are able to create their own enclaves but without direct access to the provisioning key and local report key, we have to define part of "attacker's process" within the honest process.

Within these honest processes, the attacker will be able to get a ***quote*** for their ***own enclaves*** for remote attestation, and it is signed with the provisioning key. The report data field in the quote is a public key, whose corresponding ***private key*** is ***known*** to the attacker. Moreover, the attacker will also be able to get a ***local attestation report*** for their own enclaves, and it is signed with the local report key. The private key of the public ***Diffie-Hellman key*** stated in the report is ***known*** to the attacker. That means, attackers can do anything they want ***in the name of*** their own enclaves, including but not limited to create ***malicious Decent Servers***, and getting Decent App certificates from legitimate Decent Server for their ***malicious Decent App***.

## Verification Decompositions

Some verifications will run forever, thus, we have to decompose them into small problems, and verify them one-by-one.

### 01 Secrecy of data sent between Decent Apps (which are both listed in the AuthList)

[This verification](#vf-01-decentraauapp-secrecypv) can be finished at once, thus, no decomposition is needed.

### 02 Authenticity of data sent between Decent Apps (which are both listed in the AuthList)

[This verification](#vf-02-decentraauapp-authenticitypv) can be finished at once, thus, no decomposition is needed.

### 03 Secrecy of data sent between verified Decent Apps

[This verification](#vf-03-decentravfapp-secrecypv) can be finished at once, thus, no decomposition is needed.

### 04 Authenticity of data sent between verified Decent Apps

* [**Data authenticity (when verified Decent Apps are loaded with the same AuthList)**](#vf-04-decentravfapp-authenticity-2pv)\
	All legitimate verified Decent Apps are loaded with the same legitimate AuthList
* [**Transitive trust on AuthList**](#vf-04-decentravfapp-authenticity-1pv)\
	legitimate verified Decent Apps only accept other (verified) Decent Apps loaded with the same AuthList
	* [**Correctness of Decent Verifier**](#vf-b03-decentravrfypv)\
		A legitimate Decent Verifier should only issue certificates containing the identical AuthList as the legitimate Decent App and Verifier loaded

## Core Verifications

### [vf-01-DecentRaAuApp-Secrecy.pv](vf-01-DecentRaAuApp-Secrecy.pv)

* **Brief**: Secrecy of the data sent between two legitimate Decent Apps
* **Legitimate AuthList** : [(App-D : App-A-Name), (App-A : App-A-Name), (App-B : App-B-Name), (Revoker : Revoker-Name), (Server : Server-Name)]
* **Revocation List** : [App-D]
* **Processes**:
	* Infinite replication of **IAS processes**
	* Infinite replication of **enclave platforms**
		* Infinite replication of **Decent Servers**
		* Infinite replication of **Decent Revokers** (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent App A** Acting as a **server** receiving data (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent App B** Acting as a **client** sending data (w/ legitimate AuthList)
		* Infinite replication of **malicious enclaves** (generating RA quotes & LA reports)
		* Infinite replication of **revoked Decent App D** (generating RA quotes & LA reports)
* **Query**: Can attackers access the secret data sent by the client?
* **Query in ProVerif**:

```
query attacker(secret_msg).
```

* **Rule inserted**: < 11k
* **Estimated verification time**: < 2 min
* **Result**: :white_check_mark:
* **Report**: [result-01-AuApp-Secrecy/index.html](result-01-AuApp-Secrecy/index.html)

### [vf-02-DecentRaAuApp-Authenticity.pv](vf-02-DecentRaAuApp-Authenticity.pv)

* **Brief**: Authenticity of the data transmitted between two Decent Apps
* **Legitimate AuthList** : [(App-D : App-A-Name), (App-A : App-A-Name), (App-B : App-B-Name), (Revoker : Revoker-Name), (Server : Server-Name)]
* **Revocation List** : [App-D]
* **Processes**:
	* Infinite replication of **IAS processes**
	* Infinite replication of **enclave platforms**
		* Infinite replication of **Decent Servers**
		* Infinite replication of **Decent Revokers** (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent App A** Acting as a **server** receiving data (w/ legitimate AuthList)
		* Infinite replication of **Decent App B** Acting as a **client** sending data (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **malicious enclaves** (generating RA quotes & LA reports)
		* Infinite replication of **revoked Decent App D** (generating RA quotes & LA reports)
* **Query**: If Decent App A receives any message, is that message same as the legitimate message sent by App B?
* **Query in ProVerif**:

```
query anyMsg : bitstring;
	event(DecentAppGotMsg(enclaveA, anyMsg)) ==>
	(
		(anyMsg = legitimate_msg)
	).
```

* **Rule inserted**: < 113k
* **Estimated verification time**: < 4 hr
* **Result**: :white_check_mark:
* **Report**: [result-02-AuApp-Authenticity/index.html](result-02-AuApp-Authenticity/index.html)

### [vf-03-DecentRaVfApp-Secrecy.pv](vf-03-DecentRaVfApp-Secrecy.pv)

* **Brief**: Secrecy of the data sent between two legitimate Decent Verified Apps
* **Legitimate AuthList** : [(App-D : Verifier-Name), (Verifier : Verifier-Name), (Revoker : Revoker-Name), (Server : Server-Name)]
* **Revocation List** : [App-D]
* **Processes**:
	* Infinite replication of **IAS processes**
	* Infinite replication of **enclave platforms**
		* Infinite replication of **Decent Servers**
		* Infinite replication of **Decent Revokers** (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent Verified App E** acting as a **server** receiving data (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent Verified App F** acting as a **client** sending secret data (w/ legitimate AuthList)
		* Infinite replication of **Decent Verifier** verifying Decent Verified App E (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent Verifier** verifying Decent Verified App F (w/ legitimate AuthList)
		* Infinite replication of **malicious enclaves** (generating RA quotes & LA reports)
		* Infinite replication of **revoked enclave D** which could act as a verifier, or verified app (generating RA quotes & LA reports)
* **Query**: Can attackers access the secret data sent by the client?
* **Query in ProVerif**:

```
query attacker(secret_msg).
```

* **Rule inserted**: < 177k
* **Estimated verification time**: < 8 hr
* **Result**: :white_check_mark:
* **Report**: [result-03-VfApp-Secrecy/index.html](result-03-VfApp-Secrecy/index.html)

### [vf-04-DecentRaVfApp-Authenticity-1.pv](vf-04-DecentRaVfApp-Authenticity-1.pv)

* **Brief**: Transitive trust of AuthList
* **Legitimate AuthList** : [(App-D : Verifier-Name), (Verifier : Verifier-Name), (Revoker : Revoker-Name), (Server : Server-Name)]
* **Revocation List** : [App-D]
* **Processes**:
	* Infinite replication of **IAS processes**
	* Infinite replication of **enclave platforms**
		* Infinite replication of **Decent Servers**
		* Infinite replication of **Decent Revokers** (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent Verified App E** acting as a **server** receiving the data (w/ legitimate AuthList)
		* Infinite replication of **Decent Verified App F** acting as a **client** sending the data (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent Verifier** verifying Decent Verified App E (w/ legitimate AuthList)
		* Infinite replication of **Decent Verifier** verifying Decent Verified App F (w/ AuthList given by untrusted hosts / attackers, and it's same as App F)
		* Infinite replication of **malicious enclaves** (generating RA quotes & LA reports)
		* Infinite replication of **revoked enclave D** which could act as a verifier, or verified app (generating RA quotes & LA reports)
* **Query**:
	* If Decent Verified App E accept any peer, is the AuthList stored in peer's certificate identical to the AuthList loaded by App E?
	* If Decent Verified App F accept any peer, is the AuthList stored in peer's certificate identical to the AuthList loaded by App F?
* **Query in ProVerif**:

```
query anyMsg : bitstring,
	anyAcceptedEnc : enclaveHash, anyAcceptedEncAuls : AuthList,
	anyRevcEnc : enclaveHash, anyRevcEncAuLs : AuthList, anyRevcLs : bitstring,
	anyAulsLoaded : AuthList;
	let auLs =
	AuthListInsert(AuthListNewItem(HashEnclave(enclaveD), enclaveVrfyName),
	AuthListInsert(AuthListNewItem(HashEnclave(enclaveVrfy), enclaveVrfyName),
	AuthListInsert(AuthListNewItem(HashEnclave(enclaveRecv), enclaveRecvName),
	AuthListInsert(AuthListNewItem(HashEnclave(enclaveDecentSvr), decentSvrName), AuthListEmpty)))) in
	event(DecentAppAccPeer(enclaveE, anyAcceptedEnc, anyAcceptedEncAuls)) ==>
	(
		(anyAcceptedEncAuls = auLs)
	).

query anyAcceptedEnc : enclaveHash, anyAcceptedEncAuls : AuthList,
	anyAulsLoaded : AuthList;
	event(DecentAppAccPeer(enclaveF, anyAcceptedEnc, anyAcceptedEncAuls)) ==>
	(
		event(DecentAppInit(enclaveF, spkgen(new enclaveFKeySeed), anyAulsLoaded)) ==>
		(anyAcceptedEncAuls = anyAulsLoaded)
	).
```

* **Rule inserted**: < 300k + 740k
* **Estimated verification time**: < 13 hr + 43 hr
* **Result**: :white_check_mark:
* **Report**: [result-04-VfApp-Authenticity-1/index.html](result-04-VfApp-Authenticity-1/index.html)

### [vf-04-DecentRaVfApp-Authenticity-2.pv](vf-04-DecentRaVfApp-Authenticity-2.pv)

* **Brief**: Authenticity of the data transmitted between two legitimate Decent Verified Apps
* **Legitimate AuthList** : [(App-D : Verifier-Name), (Verifier : Verifier-Name), (Revoker : Revoker-Name), (Server : Server-Name)]
* **Revocation List** : [App-D]
* **Processes**:
	* Infinite replication of **IAS processes**
	* Infinite replication of **enclave platforms**
		* Infinite replication of **Decent Servers**
		* Infinite replication of **Decent Revokers** (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent Verified App E** acting as a **server** receiving the data (w/ legitimate AuthList)
		* Infinite replication of **Decent Verified App F** acting as a **client** sending the data (w/ legitimate AuthList)
		* Infinite replication of **Decent Verifier** verifying Decent Verified App E (w/ legitimate AuthList)
		* Infinite replication of **Decent Verifier** verifying Decent Verified App F (w/ legitimate AuthList)
		* Infinite replication of **malicious enclaves** (generating RA quotes & LA reports)
		* Infinite replication of **revoked enclave D** which could act as a verifier, or verified app (generating RA quotes & LA reports)
* **Comment**:
	* Based on previous verifications, if the attacker wants the App E and F to communicate, an identical AuthList must be given to App F. If attackers don't want them to communicate, they can just block the message. Thus, in this part, a legitimate AuthList is also given to App F.
* **Query**: If Decent Verified App E receives any message, is that message same as the legitimate message sent by App F?
* **Query in ProVerif**:

```
query anyMsg : bitstring;
	event(DecentAppGotMsg(enclaveE, anyMsg)) ==>
	(anyMsg = legitimate_msg).
```

* **Rule inserted**: < 135k
* **Estimated verification time**: < 5 hr
* **Result**: :white_check_mark:
* **Report**: [result-04-VfApp-Authenticity-2/index.html](result-04-VfApp-Authenticity-2/index.html)

## Basic Verifications

### [vf-b02-DecentRaServer.pv](vf-b02-DecentRaServer.pv)

* **Brief**: Correctness of certificates issued by Decent Server - A legitimate Decent Server should issue certificates only containing the public key and AuthList that requested by the Decent App
* **Processes**:
	* Infinite replication of **IAS processes**
	* Infinite replication of **enclave platforms**
		* Infinite replication of **Decent Servers**
		* Infinite replication of **Decent Revokers** (one type of Decent App) (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **malicious enclaves** (generating RA quotes and LA reports)
* **Query**: Will a Decent Server issue a certificate containing public key and AuthList that are different from what Decent App has requested?
* **Query in ProVerif**:

```
query anyKeyIssued : spkey, anyAuLsIssued : AuthList,
	anyKeyReq : spkey, anyAuLsLoaded : AuthList;
	inj-event(DecentSvrIssueCert(enclaveDecentSvr, HashEnclave(enclaveA), new localRepKey, anyKeyIssued, anyAuLsIssued)) ==>
	(
		inj-event(DecentAppInit(enclaveA, anyKeyReq, anyAuLsLoaded)) ==>
		(
			(anyKeyIssued = anyKeyReq) && (anyAuLsIssued = anyAuLsLoaded)
		)
	).
```

* **Rule inserted**: < 1000
* **Estimated verification time**: < 1 min
* **Result**: :white_check_mark:
* **Report**: [result-b02-Server/index.html](result-b02-Server/index.html)

### [vf-b03-DecentRaVrfy.pv](vf-b03-DecentRaVrfy.pv)

* **Brief**: Correctness of certificates issued by Decent Verifier - A legitimate Decent Verifier should issue certificates only containing the public key and AuthList that requested by the Decent Verified App, and the verifier and verified app should both hold the same AuthList
* **Processes**:
	* Infinite replication of **IAS processes**
	* Infinite replication of **enclave platforms**
		* Infinite replication of **Decent Servers**
		* Infinite replication of **Decent Verifier** (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **Decent Verified App** (w/ AuthList given by untrusted hosts / attackers)
		* Infinite replication of **malicious enclaves** (generating RA quotes and LA reports)
* **Query**: Will a Decent Verifier issue certificates containing public key and AuthList that are different from what Decent Verified App has requested? Will a Decent Verifier issue certificates to Decent Verified App that holds different AuthList?
* **Query in ProVerif**:

```
query anyNameIssued : bitstring, anyHashIssued : enclaveHash, anyKeyIssued : spkey, anyAuLsVrfyHolds : AuthList, anyAuLsIssued : AuthList,
	anyKeyReq : spkey, anyAuLsReq : AuthList;
	event(DecentVrfyIssueCert(enclaveVrfy, HashEnclave(enclaveDecentSvr), anyNameIssued, anyHashIssued, anyKeyIssued, anyAuLsVrfyHolds, anyAuLsIssued)) ==>
	(
		event(DecentVrfyAppReqCert(enclaveE, anyKeyReq, anyAuLsReq)) ==>
		(
			(anyKeyIssued = anyKeyReq) &&
			(anyAuLsVrfyHolds = anyAuLsIssued) && (anyAuLsIssued = anyAuLsReq)
		)
	).
```

* **Rule inserted**: < 2000
* **Estimated verification time**: < 1 min
* **Result**: :white_check_mark:
* **Report**: [result-b03-Vrfy/index.html](result-b03-Vrfy/index.html)
