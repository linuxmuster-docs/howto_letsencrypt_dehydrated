.. Installationsleitfaden documentation master file, created by
   sphinx-quickstart on Sat Nov  7 15:29:20 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

letsencrypt für linuxmusternet einrichten
==========================================

Das Paket linuxmuster-dehydrated stellt eine Möglichkeit dar, einen linuxmuster.net Server
auf einfache Weise mit einem LetsEncrypt SSL-Zertifikat zu versorgen.

linuxmuster-dehydrated basiert auf dem Projekt `dehadrated von Lukas Schauer <https://github.com/lukas2511/dehydrated>`

Das vorliegende Dokument erläutert di Einrichtung von linuxmuster-dehydrated. 


Installation auf dem Server
===========================

Voraussetzung für die Verwendung von linuxmuster-dehydrated ist, dass der lkinuxmuster-Server 
aus dem Internet unter dem Servernamen, für den das Zertifikat erstelltz wedrden soll auf Port 
80 erreichbar ist. 

Paketinstallation
-----------------

Installieren Sie das Paket mit dem Befehl 

.. code:: bash

    apt-get install linuxmuster-dehydrated

Konfiguration anpassen
----------------------

Editieren Sie im Verzeichnis ``/etc/linuxmuster-dehydrated`` die Dateien 
``config`` sowie ``domains.txt``

In der Datei ``config`` tragen Sie eine gültige Mailadresse ein und entfernen das Kommentarzeichen

.. code:: bash

    # E-mail to use during the registration (default: <unset>)
    CONTACT_EMAIL=webmaster@ihre-domain.de

In der Datei ``domains.txt`` tragen Sie den Hostnamen ein, unter dem der linuxmuster.net Server 
vom Internet aus auf Port 80 erreichbar ist:

.. code:: bash

    # Hier muss der Servername eingetragen werden,
    # unter dem  der linuxmuster.net Server aus
    # dem Internet unter Port 80 erreichbar ist.
    
    server.ihre-domain.de


Bei Letsencrypt anmelden
------------------------

Führen Sie den Befehl 

.. code:: bash
    
    linuxmuster-dehydrated --register --accept-terms

aus. 

Die Ausgabe auf der Konsole sollte in etwa so aussehen:

.. code:: bash
    17:38/0 august /etc/linuxmuster-dehydrated # linuxmuster-dehydrated --register --accept-terms
    # INFO: Using main config file /etc/linuxmuster-dehydrated/config
    + Generating account key...
    + Registering account key with ACME server..... 

Anschließend solle es außerdem ein Verzeichnis /etc/linuxmuster-dehydrated/accounts geben:

.. code:: bash

    # ls /etc/linuxmuster-dehydrated/accounts
    aHR0xxxxxxxxxxxxxYwMS5hcGkubGV0c2VuY3J5cHQub3JnL2YYYYYYYYYYYYY


Zertifikat anfordern 
--------------------

Führen Sie den Befehl 


.. code:: bash

    linuxmuster-dehydrated --cron

aus. Die Ausgabe aug der Konsole sollte etwa so aussehen:

.. code:: bash

    # linuxmuster-dehydrated --cron
    # INFO: Using main config file /etc/linuxmuster-dehydrated/config
    Processing august.qg-moessingen.de
     + Signing domains...
     + Generating private key...
    + Generating signing request...
    + Requesting challenge for august.qg-moessingen.de...
    + Hook: Nothing to do...
    + Responding to challenge for august.qg-moessingen.de...
    + Hook: Nothing to do...
    + Challenge is valid!
    + Requesting certificate...
    + Checking certificate...
    + Done!
    + Creating fullchain.pem...
    + Hook: Restarting Apache...
    * Reloading web server config apache2 [OK]                                                                                                                                    [ OK
    + Done!
    + Hook: Nothing to do...

Das Zertifikat sollte sich nun im Verzeichnis ``/etc/linuxmuster-dehydrated/cert/<servername>/`` befinden:

.. code:: bash

    # ls /etc/linuxmuster-dehydrated/certs/august.qg-moessingen.de/
    cert-1486226502.csr  cert-1486226528.csr  cert.csr  chain-1486226528.pem  fullchain-1486226528.pem  privkey-1486226502.pem  privkey.pem
    cert-1486226502.pem  cert-1486226528.pem  cert.pem  chain.pem             fullchain.pem             privkey-1486226528.pem


Einstellungen in der apache-Konfiguration
-----------------------------------------

Im Abschnitt der Apache-Konfiguration, in dem der SSL VHost konfiguriert ist, muss nun die folgende Zertifilatskette eingetragen werden. 
Bei linuxmuster.net befindet sich diese Konfiguration für gewöhnlich in der Date``/etc/apache2/sites-enabled/000-default``

.. code:: bash

    SSLEngine On

    # <servername> anpassen!
    SSLCertificateFile     /etc/linuxmuster-dehydrated/certs/<servername>/cert.pem
    SSLCertificateKeyFile  /etc/linuxmuster-dehydrated/certs/<servername>/privkey.pem
    SSLCertificateChainFile    /etc/linuxmuster-dehydrated/certs/<servername>/chain.pem
    SSLCACertificateFile    /etc/linuxmuster-dehydrated/certs/<servername>/fullchain.pem

    # Diese Einstellungen sind optional, aber empfehlenswert
    SSLProtocol             all -SSLv2 -SSLv3
    SSLHonorCipherOrder     on
    SSLCipherSuite          ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA


Anschließend kann man den apachen neu starten.


Index 
-----

* :ref:`genindex`
* :ref:`search`

