Adding worker nodes to the OCP 4 UPI cluster existing 24+ hours

Environment
Red Hat OpenShift Container Platform 4.5 and below
Issue
Red Hat OpenShift Container Platform 4.x cluster has been installed some time ago (1+ days ago) and additional worker nodes are required to increase the capacity for the cluster.
An attempt to add a worker ends up with a failure due to Certificate signed by Unknown Authority.
Resolution
In general, a new worker has to be deployed in the same way as during the installation process (for example creating RHCOS machines during installation on bare metal) but using updated worker.ign file.
The directory with the manifest and Ignition config files that were used for the installation is required.
Below are MANUAL and AUTOMATED approaches, in both cases create some temporary directory (IMPORTANT) where one has put the worker.ign file before starting this procedure.
At the end, CSR has to be approved.
For OCP 4.6+, RHCOS now uses Ignition spec v3 as the only supported spec version of Ignition. Please refer to this article for OCP 4.6+ clusters.
MANUAL process:

1. Retrieve the certificate from the api-int.${domain}:22623 and save in a temporary file (for example api-int.pem)

Raw
$ URL=api-int.${domain}
$ openssl s_client -connect ${URL}:22623 -showcerts </dev/null 2>/dev/null|openssl x509 -outform PEM > api-int.pem
NOTE: the certificate is a block of text starting with BEGIN CERTIFICATE and ending at END CERTIFICATE.

2. Encode the Certificate using base64 with --wrap=0 option.

Raw
$ base64 --wrap=0 ./api-int.pem 1> ./api.int.base64
3. Just in case, make a backup of worker.ign and then replace ignition.security.tls.certificateAuthorities[0].source with a new value obtained in the previous step.

Raw
$ cp ./worker.ign ./worker.ign.backup
4. Put the content of api-int.base64 file to the worker.ign file and place it as below instead of $VALUE. You also need to replace the $MCP_NAME with the name of the machine config pool used for the worker node, usually is named worker or metal-worker. You can double check that getting the output of the 'oc get machineconfigpool' or check the description of one of the current active nodes.

Raw
{"ignition":{"config":{"append":[{"source":"https://api-int.<cluster.domain>:22623/config/$MCP_NAME","verification":{}}]},"security":{"tls":{"certificateAuthorities":[{"source":"data:text/plain;charset=utf-8;base64,$VALUE","verification":{}}]}},"timeouts":{},"version":"2.2.0"},"networkd":{},"passwd":{},"storage":{},"systemd":{}}
5. Upload the updated worker.ign file to the HTTP server in use for the Red Hat OpenShift Container Platform 4.x Cluster Deployment and start the worker machine.# openshiftaddingnodes
