A library for validating and generating [URL="https://en.wikipedia.org/wiki/Digital_Signature_Algorithm"]Digital Signatures[/URL] in World of Warcraft.

The Digital Signature Algorithm provides a way of ensuring data received from another player originated from the addon developer and was not modified as it was passed between players.
If you want a technical explanation of how it works you can read [URL="http://csrc.nist.gov/publications/fips/fips186-3/fips_186-3.pdf"]FIPS PUB 186-3[/URL].

The maths is complicated but the result is fairly basic, you generate a public key and a private key, you hard-code the public key into your addon, then using the private key you can sign a string of data (any length or content) and then send that data along with the generated signature to users of the addon in-game, users can use the public key to validate that the signature matches the received string, if either the signature or the string were altered by even a single bit, it will result in a failed validation.

[B]Key Generation using OpenSSL:[/B]
First get yourself a copy of [URL="http://www.openssl.org/"]OpenSSL[/URL], you may need to use it a lot or just once to generate the keys.
Once you have it compiled (Windows users can find pre-compiled binaries [URL="http://slproweb.com/products/Win32OpenSSL.html"]here[/URL]) input the following commands to generate a key pair.
[CODE]openssl dsaparam -out dsaparam.pem 1024
openssl gendsa dsaparam.pem -out dsa_priv.pem
openssl dsa -in dsa_priv.pem -pubout -out dsa_pub.pem[/CODE]
Depending on the nature of your system you may want to pick a smaller key length, currently even 128 bit keys still take significant effort to crack.
The type of the content you're sending should be the first factor you account for, other factors include how frequently the users will need to do validations and whether or not the validation needs to be done quickly or if can be run in the background.

[B]Extracting Key Values from the generated key files:[/B]
Once you have generated your key files you need to pull the actual values out of them.
First copy the contents of one of the generated pem files into an [URL="http://lapo.it/asn1js/"]ASN.1 decoder[/URL].
Decode the structure, and depending on which file you selected copy the values (in hex) into your script, use the code in DSA_test() as a reference for how to load them.
[U]Public keys[/U] contain 4 integers, 3 grouped together in a sequence, these are (in order) p, q and g, the remaining value is y, stored in a bit-string.
[U]Private keys[/U] contain 6 integers, the first of which is a zero, the remaining 5 are (in order) p,q,g,y and finally x (should never be packaged or stored anywhere publicly accessible).

[B]Signature Generation Using OpenSSL:[/B]
Signing a string follows basically the same method, these commands in sequence will take a string (payload.txt) and use the private key file to generate a signature file (sigfile.txt).
[code]openssl dgst -dss1 -sha256 -out sigfile.bin -sign dsa_priv.pem payload.txt
openssl enc -base64 -in sigfile.bin -out sigfile.txt[/code]
Use the same method you used for extracting the key values to get the signature values from the sigfile, the structure just contains the two integers r and s (in that order).

[B]In-game Signature Generation:[/B]
If you plan on generating signatures in-game you need the public key value x available in-game, this value should not be packaged with the public application, but can be extracted from the private key file.
See DSA_test() for an example of signing.