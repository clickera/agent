# DSTU4145 Agent

[![Build Status](https://travis-ci.org/dstucrypt/agent.svg?branch=master)](https://travis-ci.org/dstucrypt/agent)
[![codecov](https://codecov.io/gh/dstucrypt/agent/branch/master/graph/badge.svg)](https://codecov.io/gh/dstucrypt/agent)
[![npm module](https://badge.fury.io/js/agent.svg)](https://www.npmjs.org/package/agent)
[![dependencies](https://david-dm.org/dstucrypt/agent.png)](https://david-dm.org/dstucrypt/agent)

# Usage

## Sign and encrypt file for tax office email gate.

Ready to send to tax office email gate. Would include data, signuture, transport headers with email to send response to among other things:

    node index.js --sign --crypt  otrimano.cer \
                --tsp \
                --key Key-6.dat:password \
                --cert cert.sign.der --cert cert.cryp.der \
                --input zvit.xml --output zvit.xml.sign.enc \
                --email ilya.muromec@gmail.com

Note: name of input file AND name of email attachment matters for processing server.

Filename format for tax office is following:

    '15 01 3225000000 F01 033 05 1 00 0000001 5 12 2015 15 01.xml'
                            tax period code --^
                 tax period length in months  --^
                                        tax year -- ^
                  code of tax office and region again --^---^
                                      ^-- document increment id
                                   ^-- document amendment version
                                 ^-- document state
                      ^-- for code and version
           ^-- your tax id
        ^-- code of tax office
     ^-- code of tax region

## Sign file (czo.gov.ua/verify)

When `--no-tax` option is specified in command line, both transport header and `UA_SIGN1` prefix would be ommited, producing raw ASN1 file in CMS (almosrt) format. Email option is ignored in this mode, as no transport header is writte. Makes most sense to sign contracts and such:

    node index.js --sign \
                --key Key-6.dat:password \
                --cert cert.sign.der \
                --input text.pdf --output text.pdf.p7s \
                --no-tax \
                --tsp
                
Example commandline for privatbank keys:
    
    node index.js --sign \
                --key pb_1234567890.jks:password \
                --input text.pdf --output text.pdf.p7s \
                --no-tax \
                --tsp
    

## Write detached signature

When `--detached` option is specified in command line, resulting file would only contain signuture. Signed data would not be included. Makes most sense with `--no-tax` present and `--email`` ommited. This is not compatible with tax office email gate:

    node index.js --sign \
                --key Key-6.dat:password \
                --cert cert.sign.der \
                --input zvit.xml --output zvit.xml.sign \
                --detached --no-tax

## Load key from jks store (privatbank)

Since version 0.4.40 it's possible to use use jks files with agent. Since jks file format contains number of keys at the same time, with first key being electronic stamp (not a personal key), agent has support of `--role` option in commandline. Possible values are: 

 - personal - certificate belongs to natural person and has no record of any corporate entity;
 - fop (fizychna osoba pidpryjemets) - certificate belongs to natural person registered as private entrepreneur, technically this means that personal code (10, 9 or 8 digit DRFO) matches corporate code (EDRPOU);
 - director - certificate either belongs to FOP or natural person that can sign on behalf of corporate entity, technicall this means that corporate code either matches drfo or drfo code is present and corporate code does not belong to natural person;
 - stamp - certificate belongs to corporate entity itself, not natural person;
 - other - personal code is present but does not match corporate code (relaxed version of director);
 - exact personal code (either DRFO or passport number for religious people) to match. should be 10, 9 or 8 characters long
 
 Example:
 
     node index.js --sign \
                --key Key-6.dat:password \
                --cert cert.sign.der \
                --input zvit.xml --output zvit.xml.sign \
                --detached --no-tax \
                --role stamp

## Load key from nonencrypted store

    node index.js --sign \
                --key Key-6.raw \
                --cert cert.sign.der \
                --input zvit.xml --output zvit.xml.sign

## Encrypt only

Notice, that both certificates are specified. This is implementation requirement, not really needed in code.

    node index.js --crypt  otrimano.cer \
                --key Key-6.dat:password \
                --cert cert.sign.der --cert cert.cryp.der \
                --input zvit.xml --output zvit.xml.sign.enc

## Unwrap

To unwrap and decrypt incoming messages, use `--decrypt` command-line switch. Notice that certificate is not required here.

    node index.js --decrypt \
                --key fop_acsk.raw.der \
                --input incoming.encrypted \
                --output clear

## TSP

To add secure timestamp, use `--tsp` command-line switch. Secure timestamp is mandatory for long-term storage since November 7th 2018.

     node index.js --sign \
                --tsp \
                --key Key-6.raw \
                --cert cert.sign.der \
                --input zvit.xml --output zvit.xml.sign


## Agent mode

Signer can operate as out-of-process agent. One process would load key storage and listen to local socket,
while other would pass data to be signed to it.

Example:

    node index.js --agent \
                  --key Key-6.dat:password \
                  --cert cert.ipp.sign --cert cert.ipp.cryp &
    node index.js --connect \
                  --sign \
                  --input zvit.xml --output zvit.xml.sign


Agent mode is available for encrypt and unwrap operations as well:

    # start agent as specified above
    node index.js --decrypt --connect --input encrypted.pkcs7 --output clear


## Key unwrapper

Normaly keys are stored inside encrypted file called Key-6.dat that requires password
to be decrypted every time it is being loaded.

However you can remove this protection from and store raw version of file.

    node index.js --unprotect --key Key-6.dat:password --output fop_acsk.raw.der
    node index.js --unprotect --key Key-6.dat:password > fop_acsk.raw.pem

Notice that without `--output` argument, private is outputed to standard output in PEM form.

## Notes

Starting from jkurwa version 0.4.20, agent can read documents created by "ME.DOC" software.

# Reference

This app is able to sign and encrypt pre-crafted tax reports. Tax report format specifications:

* http://opz.at.ua/index/struktura_fajla_xml/0-57 -- Filename structure and file general file format
* http://sfs.gov.ua/data/material/000/006/58768/Forms1.htm -- Tax form specification
* http://github.com/muromec/taxes-ua -- Tax difference calculator
