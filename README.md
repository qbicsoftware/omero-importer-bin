# Omero importer CLI
A description how to build a slim JAVA omero-importer CLI, without installing OMERO.server.

## Motivation

During the time of this writing, OMERO provides only one way to install an OMERO CLI that provides import functionality to OMERO, and this
is via an complete OMERO.server installation. 

As we think, this is quite some batteries included to ship a simple CLI to import images to OMERO, we decided to extract the necessary classes and provide
a more slim artifact.

You can either download the executable JAR directly from this repository, or follow the manual installation instruction.

## Manual installation

### Download the OMERO.insight app

OMERO related artifacts can be found on their [download mirror](https://downloads.openmicroscopy.org/omero). 

For version 5.4.6, download the archive directly with `wget`:
```
wget https://downloads.openmicroscopy.org/omero/5.4.6/artifacts/OMERO.importer-5.4.6-ice36-b87-linux.zip
```

### Unzip binary

Unzip the content into an directory of your choice. We refer to this directory with the placeholder `$DESTINATION_DIR`.

```
mkdir ~/temp_omero_content
export DESTINATION_DIR=~/temp_omero_content
unzip OMERO.importer-5.4.6-ice36-b87-linux.zip -d $DESTINATION_DIR
cd $DESTINATION_DIR
```

The content should look similar to this:

```
$ ls -1
INSTALL.txt
OMEROimporter_unix.sh
config
importer-cli
libs
omero.insight.jar
```

We are interested in particular in the libraries shipped in the `libs` folder.

### Prepare content 

We need to unzip all JARs in the `lib` folder first and collect them in one directly that will be the input for the final (FAT) JAR.

Create an output directory at the location of you choice. For macOS, you can execute the following command. Linux-based systems should work similar.

```
$ ls -1 | xargs -I {} unzip {} -d $OUTPUT_DIR
```

This extracts all content into one folder. 

Next, we need to move the META-INF directory present in the output and temporarily store it somewhere. We will need some of the content later again. Do not delete it!

### Create executable JAR

Now make sure, that your current working directory is the `$OUTPUT_DIR` with all the extracted JARs and removed META-INF folder. 

```
jar -cfve omero-importer.jar ome.formats.importer.cli.CommandLineImporter ./
```

This creates a fat executable JAR called `omero-importer.jar` in the working dir. Confirm, that the JAR creation worked and that the main entry class can be found:

```
$ java -jar omero-importer.jar -help
16:22:20.955 [main] INFO  ome.formats.importer.ImportConfig - OMERO Version: 5.4.6-ice36-b87
16:22:20.972 [main] INFO  ome.formats.importer.ImportConfig - Bioformats version: (unknown version) revision: (unknown revision) date: (unknown date)

 Usage:  importer-cli [OPTION]... [path [path ...]]...
   or:   importer-cli [OPTION]... -

Import any number of files into an OMERO instance.
If "-" is the only path, a list of files or directories
is read from standard in. Directories will be searched for
all valid imports.

Session arguments:
  Mandatory arguments for creating a session are 1- either the OMERO server hostname,
username and password or 2- the OMERO server hostname and a valid session key.
  -s SERVER	OMERO server hostname
  -u USER	OMERO username
  -w PASSWORD	OMERO password
  -k KEY	OMERO session key (UUID of an active session)
  -p PORT	OMERO server port (default: 4064)

Naming arguments:
All naming arguments are optional
  -n NAME				Image or plate name to use
  -x DESCRIPTION			Image or plate description to use
  --name NAME				Image or plate name to use
  ...
  ...
```

### Finalize JAR

Now we are still missing some OS dependent libraries. These are located in the previously moved `META-INF` folder under the lib directory. Go to this directory now.

You can remove everything but the folders `services` and `libs`. 

Now change the directory one level up with `cd ..`. 

Lastly, we update the JAR file and add these important dependencies with:

```
jar -uf path/to/omero-importer.jar META-INF
```

That's it. Have fun to use the Java-based OMERO impoerter CLI.


