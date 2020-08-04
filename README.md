# Protected Digital Identity - A Proposition

_Preliminary proposition_

This document aims to propose a solution to the problem of our digital identity protection. From a digital point of view, our identity often rely only on our email address that our correspondents have to decide whether or not they trust. The initial assertion is here to assume that the email address is a very weak and unreliable way to determine our digital identity.

In addition, taking the example of Switzerland, projects of digital identities are currently discussed at the political level. It seems, for now, that the swiss authorities are not disposed to take this responsibility and seem to currently favor a solution in the private sector. Such a solution, relying on private interests to ensure our digital identity, seem very dangerous as private companies have only one goal : making as most money as possible while offering as less as possible for their clients.

This preliminary documents proposes a solution that does not rely on any central authority to ensure the protection of our digital identity. You are welcomed to contribute to this proposition.

> The proposed solution is currently formulated in term of commands used in the context of a UNIX-like system.

# Context

The problem of the identity of someone you meet for the first time is not new and many solution have been brought. The proposed solution takes its inspiration in the HTTPS (SSL/TLS) protocol.

# Digital Identity : Digital Name

In the first place, the email address are not reliable to determine our digital identity as they are under the control of company that can fail, disappear or even fire you, leaving you without the email address you use to use and through which your contacts know you.

This solution propose to replace your email address by a simple string as your digital identity : your digital name. This string rely on the following information :

    * First name(s)
    * Last name(s)
    * Date of birth
    * The state/region/canton of birth
    * The country of birth

For example :

    * Guillaume Jean-Baptiste
    * Le Gentil
    * September 12, 1725
    * Normandie
    * France

The proposed solution tries to minimize the exposition of these information as they are still, unfortunately, used by entities to check you identity. The digital name starts by contracting the previous information as follows :

    guillaume.jean-baptiste.le-gentil.17250912.normandie.france

using dots to separates the parts, minus to join element inside parts, all in lowercases. In order to keep these information hidden, the SHA-256 is used to compute the [hash](https://en.wikipedia.org/wiki/Hash_function) of the string is computed with the command (UNIX-like) :

    $ echo "guillaume.jean-baptiste.le-gentil.17250912.normandie.france" | sha256sum

which provides the hash string :

    1bc50ad29e7a355bbcdc669db6af1b9136dd2764c90f5eb310aac3c82aee7ad4

To reduce the size, only the 16-first digits (chars) are kept, group by four. This 16-first digits are used to build the digital name :

    guillaume.jean-baptiste.le-gentil.1bc5-0ad2-9e7a-355b

The usage of the information on birth are used to avoid collision leading to homonym having the same digital name.

In other words, you are in charge of building your digital name without any external validation.

# Digital Identity : Digital Protection

In order to be able to protect your identity, that is being able to prove the ownership of a digital name, the [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) is considered allowing asymmetric encryption.

The only thing you have to add to your digital name to protect it is a RSA keys pair associated with a validity time range.

To create a pair of RSA key, the following command is used (OpenSSL, UNIX-like) :

    $ openssl genpkey -algorithm RSA -out private.pem -pkeyopt keygen_bits:256

which leads to the creation of a file named 'private.pem' containing your secret key that should look like this :

    -----BEGIN PRIVATE KEY-----
    MIHCAgEAMA0GCSqGSIb3DQEBAQUABIGtMIGqAgEAAiEA3UgBMJzO1ov5Yn5eJDdJ
    ti6tCftooZ3nW4VRsW9Sg1MCAwEAAQIhAJvj0ELJFcZ8EhLbZ8Mn2BrauGtzyp6J
    cvyvl0jfyGrhAhEA/cOXGGdnOCB4GrHs8byO6wIRAN87JNDZ2P+fnq5NnULh0zkC
    ECDN4AJvm5BN4jjRN2goj/ECEDWTa02Yy0TmmV36EMFJk7kCEAvLkhesNlsf9D08
    gR1uOrg=
    -----END PRIVATE KEY-----

> In this example, specifying 256 as a bit count is very weak. In your case, you should go with 2048 as a minimum, 4096 or 8192 to be more careful.

Your private key has to remains secret and only know by you. Do never show it or give it to anyone, even if you trust the person.

In order to get the second RSA key, the public key, simply use the command (UNIX-like) :

    $ openssl rsa -pubout -in private.pem -out public.pem

where you provide your created secret key to obtain the public one 'public.pem' with a content similar to :

    -----BEGIN PUBLIC KEY-----
    MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAN1IATCcztaL+WJ+XiQ3SbYurQn7aKGd
    51uFUbFvUoNTAgMBAAE=
    -----END PUBLIC KEY-----

This public key is the key you will share with everyone (it has not to be kept secret). As a summary, your digital identity consist in three public elements :

    * your digital name (that you built on your own)
    * your public key
    * the validity period of the key (e.g. one year after its creation)

and one secret element :

    * your secret key corresponding to the public one

Your secret key is then the only secret element you have to take care of (a continued attention).

# Digital Identity : Verification

> For the rest of the documents, trying to keep things clear, the 'I' (the writer) refers to the person trying to protect its digital identity while the 'you' (the reader) refers to the person that try to verify that digital identity.

I have a digital identity that I claim and, as you are not convinced, you want to verify that I am the person able to claim that identity.

Two levels of verification are proposed, in an order that is justified later.

## First Level : Two-Side Verification

This verification only imply you and me. It is also assumed that you don't have any digital identity (and no keys) on your side.

You start the verification by creating ex-nihilo and file containing only random bytes with the command (UNIX-like) :

    $ openssl rand -out random-file.org 32

that creates a 32-bytes file named 'random-file.org'. You use my public key (my_public.pem) I provided you to encrypt this file with the command (UNIX-like) :

    $ openssl rsautl -encrypt -inkey my_public.pem -pubin -in random-file -out random-file.enc

then you send me the encrypted file 'random-file.enc'. Because its encrypted with my public key, my private key (my_private.pem) is the only one able to decrypt it, that I do using the command (UNIX-like) :

    $ openssl rsautl -decrypt -inkey my_private.pem -in random-file.enc -out random-file.dec

which allows me to discover the content of your random file (random-file.dec). I then use my private key to sign the random file I just decrypt using the command (UNIX-like) :

    $ openssl dgst -sha256 -sign my_private.pem -out random-file.sig random-file.dec   

creating the signature file named 'random-file.sig'. I then send you the signature file that you check using my public key and your original of the random file using the command (UNIX-like) :

    $ openssl dgst -sha256 -verify my_public.pem -signature random-file.sig random-file

If the outcome of the command is 'Verified OK', the verification is a success, otherwise, there is someone between us.

## Second Level : Third-Party Protectors

The previous two-side verification, from your point of view, only ensure that the presumed 'me' has the secret key corresponding to the public key you received _a priori_ from me. If someone is in the middle of us, he could have replaced my public key by its own one (for which he has the corresponding secret key) without you and me noticing it, corrupting the previous verification.

To tackle this problem, the concept of third-party protectors (TPP) is introduced. It is a small set of persons you know that keep a copy of your public key (and its validity dates). You need, just after the creation of your public key, to have a way to give them securely a copy of it (through a physical meeting or a phone verification).

These TPP are then used by you to check that the public key I presumably gave you is effectively my public key.

In order to allow anyone to be my TPP, two verification procedures are proposed : one without technical aspects and a more formal one.

> Formally, the following check with third-parties should be made first. As TPP are volunteers, this second level verification should be left to cases where a strong verification is mandatory.

### Third-Party Protectors of Type A (TPP-A)

In such a case, my TPP of type A is clearly not a geek. So, he cannot perform advanced verification procedure.

To verify the authenticity of my presumed public key, simply pack in an email to one of my TPP-A, that you randomly choose, my presumed public key, its validity dates and my digital name.

By simple visual comparison, if he answer you that you have the same information about me as he does, the verification is a success. Otherwise, someone is in the middle of us.

### Third-Party Protectors of Type B (TPP-B)

For the TPP of type B, a preliminary procedure is required between him and me (typically just after I created by public key).

I send him my public key in a secure way asking him to sign it with its own private key as follows (UNIX-like) :

    $ openssl dgst -sha256 -sign tppb_private.pem -out my_public_key_signed_by_tppb.sig my_public_key.pem

he sends me back the signature he computed with its private key and drops my public key.

To verify the authenticity of my presumed public key, you simply provides to the TPP-B you chose my digital name and the validity date of my key. He tells you if he agrees with the dates along with its own public key.

You then simply ask me the signature he made for my public key that I send you. You simply verify the signature as follows (UNIX-like) :

    $ openssl dgst -sha256 -verify tppb_public_key.pem -signature my_public_key_signed_by_tppb.sig my_public_key.pem

If the outcome of the command is 'Verified OK', you have the confirmation that the public key is mine. Otherwise, someone is in the middle of us.

# License and Copyrights

*protected-digital-identity*
CC BY-NC-SA - Nils Hamel

# Requirements

In order to create and protect a digital identity, you will need the following tools :

    * At least one active email address
    * [OpenSSL](https://www.openssl.org) library and tools
    * An SHA-256 computation tool

It is recommended to use a password manager that allows to import and encrypt files along with your passwords such as :

    * [KeepassXC](https://keepassxc.org)

Of course, you will also need a bit of motivation.
