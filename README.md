diff --git a/README.md b/README.md
index eb1b3c8dd812280f6dd098f795a02d0465c0e1e7..826f91c834ba928469ece48ba8d99287f696059c 100644
--- a/README.md
+++ b/README.md
@@ -108,25 +108,75 @@ GLOBAL OPTIONS:
 ```
 
 Example:
 
 ```
 INTERVAL=10s /usr/sbin/openldap_exporter --promAddr ":8080" --config /etc/slapd/exporter.yaml
 ```
 
 Where `exporter.yaml` looks like this:
 
 ```yaml
 ---
 ldapUser: "cn=monitoring,cn=Monitor"
 ldapPass: "sekret"
 ```
 
 NOTES:
 
 * `ldapNet` allows you to configure `tcp` or `unix` socket connections to your co-located OpenLDAP server.
 * `webCfgFile` can be used to provide authentication and TLS configuration for the [prometheus web exporter](https://github.com/prometheus/exporter-toolkit/tree/master/web).
 
 ## Build
 
 1. Install Go 1.20 from https://golang.org/
 2. Build the binaries: `make build`
+
+The build output is placed in the `dist/` directory. The Linux amd64 archive produced by
+the GitHub Actions workflow contains an `openldap_exporter` binary you can copy to the
+target host.
+
+## Run
+
+You can run the exporter directly after downloading the binary or building it from
+source:
+
+```
+$ ./openldap_exporter --ldapAddr "localhost:389" --promAddr ":9330"
+```
+
+Set the command-line flags and environment variables described above to match your
+deployment. For production environments, place the binary somewhere on your `PATH`
+(for example, `/usr/local/bin/openldap_exporter`) and ensure the user running the
+service has permission to access the OpenLDAP monitor interface.
+
+### Running as a systemd service
+
+To manage the exporter with `systemd`, create a service unit such as
+`/etc/systemd/system/openldap_exporter.service` with the following contents:
+
+```
+[Unit]
+Description=Prometheus OpenLDAP exporter
+After=network.target
+
+[Service]
+Type=simple
+User=openldap-exporter
+Group=openldap-exporter
+ExecStart=/usr/local/bin/openldap_exporter --config /etc/openldap_exporter/config.yaml
+Restart=on-failure
+
+[Install]
+WantedBy=multi-user.target
+```
+
+Adjust the `User`, `Group`, `ExecStart`, and configuration path to suit your
+environment. After creating the unit file, reload the `systemd` daemon, enable the
+service, and start it:
+
+```
+$ sudo systemctl daemon-reload
+$ sudo systemctl enable --now openldap_exporter.service
+```
+
+This will ensure the exporter starts automatically on boot and restarts on failure.
