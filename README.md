# Goose Protocol Parser For Zeek IDS
GOOSE is a communication protocol defined in the IEC61850 standard. It is used by Intelligent Electronic Devices (IEDs) in electrical substations to facilitate information exchange between devices. A GOOSE parser has been developed to enable detailed analysis of the transmitted data and allow rule-based identification of anomalies related to cybersecurity attacks. It is compatible with an older instance of Zeek Network Security Monitor (v2.6).

## System Requirements
In general, the GOOSE parser can run on any system that supports Zeek. In our setup, it was tested successfully in a virtual machine environment with the following configuration.

|Component|Setting|
|---|---|
|Operating System|Ubuntu 18.04|
|RAM|4 GB|
|Processor|3.5 GHz|
|Disk Space|20 GB|

## Installation
The GOOSE parser is built upon the framework provided by the Zeek Network Security Monitor. Formerly known as Bro, Zeek is an open source IDS which allows comprehensive network analysis. The GOOSE parser, downloadable as a patch, has to be applied to a compatible version of Zeek. The installation steps are shown below.

### Zeek Installation
1.	Install the required dependencies for Zeek listed in the official website:
https://docs.zeek.org/en/current/install/install.html#required-dependencies
2.	Clone the Zeek repository with the following command:
<br/>`git clone --recursive https://github.com/zeek/zeek`

### Patch Installation
1.	Switch to a snapshot of the Zeek repository that is compatible with the GOOSE parser.
```
cd <zeek_dir>/
git checkout aff3f4
git submodule update --init --recursive
cd ..
```
2.	Apply the GOOSE parser as a patch. The whitespace warnings may be ignored.
```
git clone https://github.com/smartgridadsc/Goose-protocol-parser-for-Zeek-IDS
mv Goose-protocol-parser-for-Zeek-IDS/patch/goose_parser.patch zeek/ && cd zeek
git apply --reject --whitespace=fix goose_parser.patch
```
<br/>Note: Please refer to the User Guide in the [doc/](doc) folder for the full list of modified files.

3.	Build and install from source. Some commands may require root privileges.
```
./configure
make
make install
```

## Usage
The ADSC Github repository contains sample GOOSE trace files. These traces were generated as part of a research project in GOOSE communication within a typical substation and are available for download from this link: https://github.com/smartgridadsc/IEC61850SecurityDataset

Three sample trace files have been provided in /scripts/base/protocols/goose/.
- Sample_Script_A.bro -> Prints and logs GOOSE packets
- Sample_Script_B.bro -> Checks if stNum value rolls over correctly
- Sample_Script_C.bro -> Checks if stNum/sqNum values are set accordingly when dataset is updated

A trace file can be analysed in Zeek from the terminal with the following commands:
```
cd <zeek_dir>
sudo ./build/src/bro –r <trace_file> ./scripts/base/protocols/goose/<script_name>.bro
```
When executing the above commands with Sample_Script_A.bro, a subset of the GOOSE data fields relevant for cybersecurity analysis will be printed to the console and logged into ‘goose.log’.

## Generated Event
On successful parsing of a packet, an event is raised with the following signature, accessible from user-defined scripts.
<br/>`event goose_packet_event(p: goose_records::gcp)`<br/>
The input parameter p, of type, goose_records::gcp is a Zeek record and defines the following fields in init-bare.bro:
|Description|Field Name|Zeek Datatype|
|---|---|---|
|Packet timestamp|packet_ts|double|
|Source MAC Address|src_mac|string|
|Destination MAC Address|dest_mac|string|
|Status Number|stNum|count|
|Sequence Number|sqNum|count|
|gocbRef|packet_type|string|
|*Numeric data entries|data_values|vector of double|
|String data entries|string_values|vector of string|

*All data entries of type int, bool and float will be converted to double when passed from the event engine to the scripting layer.

## Further Reading
Please refer to the [doc/](doc) folder for the parser design and contact information.
