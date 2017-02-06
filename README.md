fips-compliant-vault
====================

A FIPS 140-2 level 1 compliant implementation of a password vault
for Red Hat JBoss EAP 6 that works across operating systems.  This
strictly means that EAP is using FIPS 140-2 level 1 certified
libraries to provide the cryptographic functions needed to mask
sensitive strings in the EAP configuration files.  Red Hat JBoss
EAP itself is not FIPS certified, but it's able to use those certified
libraries.  This implementation takes advantage of the (Legion of
the Bouncy Castle pure java FIPS 140-2 Level 1 certified library)[http://www.bouncycastle.org/fips-java/].
Huge thanks to them for their efforts getting their implementation
certified!  If you find their libraries useful, please contribute
to their ongoing efforts to maintain certification.

Disclaimer
----------

Cryptography is hard.  Doing it well requires more expertise or
special knowledge beyond the available documentation for the
underlying java and bouncy castle interfaces.  If you find a hole,
please let me know.  Seriously.  Also, if you're masking the keystore
password, be aware that this is more obfuscation than making the
password secure.  If you truly want a secure password, investigate
the alternative 'password commands' available to use host executables
or custom class files to retrieve the keystore password.

Build and Install the Vault
===========================

Configure Tools to Build
------------------------

This should build on all operating systems that support both java
and maven.  Please make sure that you have that tooling in place.
In addtion, you need to get the certified Legion of the Bouncy
Castle Java FIPS library from the (Bouncy Castle Java FIPS page)[http://www.bouncycastle.org/fips-java/].
After the click-through acknowledgement, download the provider
bc-fips-1.0.0.jar file.  Next, add that jar file to your local maven
repository.  In a terminal window, type the following command:

    mvn install:install-file -Dfile=bc-fips-1.0.0.jar \
        -DgroupId=org.bouncycastle -DartifactId=bc-fips -Dversion=1.0.0 \
        -Dpackaging=jar

Build and Deploy the Custom Vault
---------------------------------

Building the custom vault is simple.

    git clone https://github.com/rlucente-se-jboss/fips-compliant-vault.git -b bcfips
    cd fips-compliant-vault
    mvn clean package

The results are assembled into a distribution zip file available
in `target/fips-compliant-vault-1.0.0-dist.zip`.  To deploy the
vault, unzip the distribution file into your `$JBOSS_HOME` folder:

    export JBOSS_HOME=/path/to/java-eap-6.4
    unzip -q target/fips-compliant-vault-1.0.0-dist.zip -d $JBOSS_HOME
    chmod a+x $JBOSS_HOME/bin/fips-vault.sh

Initialize and Populate the Vault
=================================

There are so many options and ways to setup the password vault.
This implementation also makes rekeying as painless as possible.
These instructions cover all of the options available.

Interactively Initialize an Empty Vault
---------------------------------------

To create the necessary files and populate the vault with masked
strings, please use the script without any arguments:

    $JBOSS_HOME/bin/fips-vault.sh

This will prompt the user to provide the needed parameters and store
sensitive strings in the vault.  An example session is below:

    bash-3.2$ cd $JBOSS_HOME
    bash-3.2$ bin/fips-vault.sh
    =========================================================================
    
      JBoss Vault
    
      JBOSS_HOME: /Users/rlucente/demo/eap-6.4/jboss-eap-6.4
    
      JAVA: java
    
    =========================================================================
    
    **********************************
    ****  JBoss Vault  ***************
    **********************************
    Please enter a Digit::   0: Start Interactive Session  1: Remove Interactive Session  2: Exit
    0
    Starting an interactive session
    Enter directory to store encrypted files: /Users/rlucente/demo/eap-6.4/jboss-eap-6.4/vault
    Enter Keystore URL: /Users/rlucente/demo/eap-6.4/jboss-eap-6.4/vault/vault.bcfks
    Create the keystore if it doesn't exist <y/N>: y
    
    Please enter the keystore password: 
    Please confirm the keystore password: 
    
    The salt must be at least 16 bytes in length, before base-64 encoding.
    Enter salt as a base-64 string (or ENTER for a random value): 
    
    The iteration count must be at least 1000
    Enter iteration count as a number (Eg: 2000): 1000
    
    The initialization vector must be 16 bytes in length, before base-64 encoding.
    Enter iv as a base-64 string (or ENTER for a random value): 
    Enter Keystore Alias: adminKey
    Initializing Vault
    Feb 06, 2017 5:06:50 PM org.jboss.security.fips.plugins.FIPSSecurityVault setUpVault
    INFO: FIPS000373: Generating a new admin key under alias (adminKey)
    Feb 06, 2017 5:06:50 PM org.jboss.security.fips.plugins.FIPSSecurityVault init
    INFO: FIPS000361: FIPS Security Vault Implementation Initialized and Ready
    
    *******************************************
    Copy the following <vault/> element to your
    standalone or domain configuration file to
    enable the password vault.
    *******************************************
        ...
        </extensions>
        <vault code="org.jboss.security.fips.plugins.FIPSSecurityVault" module="org.jboss.security.fips" >
          <vault-option name="ENC_FILE_DIR" value="/Users/rlucente/demo/eap-6.4/jboss-eap-6.4/vault/"/>
          <vault-option name="INITIALIZATION_VECTOR" value="DOlNP10ToARAO7VtW23Nwg=="/>
          <vault-option name="ITERATION_COUNT" value="1000"/>
          <vault-option name="KEYSTORE_ALIAS" value="adminKey"/>
          <vault-option name="KEYSTORE_PASSWORD" value="MASK-vUGBkyW1mjdK9MDDJKMEGQ=="/>
          <vault-option name="KEYSTORE_URL" value="/Users/rlucente/demo/eap-6.4/jboss-eap-6.4/vault/vault.bcfks"/>
          <vault-option name="SALT" value="nzCeFZAeXbkCNi3Vh5+oeQ=="/>
        </vault>
        <management>
        ...
    *******************************************
    
    Vault is initialized and ready for use
    Please enter a Digit::  0: Store a secured attribute  1: Check whether a secured attribute exists  2: Remove secured attribute  3: List all secured attributes  4: Exit
    0
    Task: Store a secured attribute
    
    Please enter the secured attribute value (e.g. a password): 
    Please confirm the secured attribute value (e.g. a password): 
    Enter Vault Block: keystore
    Enter Attribute Name: password
    
    ******************************************************************************
    The secured attribute value has been stored in the password vault.  Please
    make note of the following:
    ******************************************************************************
    Vault Block:keystore
    Attribute Name:password
    
    The following string should be cut/pasted wherever this password occurs in the
    EAP configuration file.  If you're changing an existing password in the vault,
    the entry in the configuration file can remain the same:
    
    ${VAULT::keystore::password::1}
    ******************************************************************************
    
    Please enter a Digit::  0: Store a secured attribute  1: Check whether a secured attribute exists  2: Remove secured attribute  3: List all secured attributes  4: Exit
    3
    Task:  List all secured attributes formatted as <vault-block>::<attribute>
    
    	keystore::password
    
    Please enter a Digit::  0: Store a secured attribute  1: Check whether a secured attribute exists  2: Remove secured attribute  3: List all secured attributes  4: Exit
    4
    bash-3.2$ exit

