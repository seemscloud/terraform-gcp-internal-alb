```terraform
provider "google" {
  project     = "seemscloud"
  region      = "europe-central2"
  credentials = "/home/taw/.gcp-creds.json"
}

terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "5.3.0"
    }
  }
}

resource "google_compute_forwarding_rule" "http" {
  name                  = "regional-http"
  load_balancing_scheme = "INTERNAL_MANAGED"
  ip_protocol           = "TCP"
  port_range            = "80"
  target                = google_compute_region_target_http_proxy.default.id
  ip_address            = google_compute_address.http.address
  network               = "projects/seemscloud/global/networks/default"
  subnetwork            = "projects/seemscloud/regions/europe-central2/subnetworks/default"
  network_tier          = "PREMIUM"
}

resource "google_compute_forwarding_rule" "https" {
  name                  = "regional-https"
  load_balancing_scheme = "INTERNAL_MANAGED"
  ip_protocol           = "TCP"
  port_range            = "443"
  target                = google_compute_region_target_https_proxy.default.id
  ip_address            = google_compute_address.https.address
  network               = "projects/seemscloud/global/networks/default"
  subnetwork            = "projects/seemscloud/regions/europe-central2/subnetworks/default"
  network_tier          = "PREMIUM"
}

resource "google_compute_address" "http" {
  name         = "regional-http"
  address_type = "INTERNAL"
  subnetwork   = "projects/seemscloud/regions/europe-central2/subnetworks/default"
}

resource "google_compute_address" "https" {
  name         = "regional-https"
  address_type = "INTERNAL"
  subnetwork   = "projects/seemscloud/regions/europe-central2/subnetworks/default"
}




resource "google_compute_region_target_http_proxy" "default" {
  name    = "regional-thp"
  url_map = google_compute_region_url_map.http.id
}

resource "google_compute_region_target_https_proxy" "default" {
  name             = "regional-https"
  ssl_certificates = [
    google_compute_region_ssl_certificate.default.self_link
  ]
  url_map = google_compute_region_url_map.https.id
}




resource "google_compute_region_url_map" "http" {
  name            = "regional-http"
  default_service = google_compute_region_backend_service.default.id
}

resource "google_compute_region_url_map" "https" {
  name            = "regional-https"
  default_service = google_compute_region_backend_service.default.id
}




resource "google_compute_region_backend_service" "default" {
  name                  = "regional-bs"
  load_balancing_scheme = "INTERNAL_MANAGED"

  backend {
    group           = "projects/seemscloud/zones/europe-central2-b/instanceGroups/http"
    balancing_mode  = "UTILIZATION"
    capacity_scaler = 1.0
  }

  health_checks = [
    google_compute_health_check.default.id
  ]
}

resource "google_compute_health_check" "default" {
  name = "regional-hc"

  http_health_check {
    port         = 80
    request_path = "/"
  }
}

resource "google_compute_region_ssl_certificate" "default" {
  name        = "regional-sc"
  private_key = <<EOT
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCc+fzvnHU5+3rF
2mk150f/rJ9IrpFORVq5yRTXCMmq87baJVdi9k1jIVUUtB26ocZk05WoPIj0NWa/
Dnk14zKCvnjRaPxIaN4kVsUMfnRajhIwUkSTzVU5Ny7n8aBDKtiaPALKgrF7Jy/9
AmahDCkbPbNWrUBpGIL03nJvqGrg+z+i9mwKi4nM1+RB9Or8zwmhBBzJOADlzQNh
psp3K6cErDN2oO17tbdAqOy2lQM9jWXoxyyZ4rT9hBMJgBpQFD1lW5WJ3OdPu0+t
2FqBDPPaE7YcwqeEDtN0YxFGOTSKX6Ju/q/ZHvXHsknxRPn3sEWQVi7pYOe6WUzA
9JyuD7wbAgMBAAECggEASSI60lN9Vg8wyt8P51oidB6zcwRJlELBmw7w06I3eprj
FM0S4ntH4KhV4DhuJVZbfPwKnH/feo8qmFag6Bc6oCknVsDn0MByxlFaqpB7mvjX
xoV9h7LBQs/P3aX3+XMLyQeucTTkhTDjraehsuRcfmGHnRlIie4ujPDaNeUsLjaF
A4cz7PP0bLhE0XlIM5/GVbfasN++HCRhY9XnLBONDegYXpbN+Q8JCQ+4rJ8iptdi
TmGsSih4oKn+SMndujmmkidcght2iR8cAVH732DdPAq1SxOTINX/R6B9wSKeI3GT
+3jCjvJ3w7d7qDERjOcQiy6tDUuI4aChDG2LQ+kOyQKBgQC9viMSttvFBYnNQsxA
zaaRrFKK7UOKzz4pFqro56ZAXaBjU2NwuAQLqtranMIwtCtUsadvSatzD/WrjRvE
Cp7nSrsKVXnoAOjaKA4FSRI+iCIKxPVsFBwGs3lgspXEe7q1bNeR0PlVDm/QqAAG
b7neclziB+n/zYd+KMZWOe4V/QKBgQDTyr+VSmXQup8+++Namb2LBIwBHRsyfGP2
Ue6d8NuKeaBy5/YSrZnkMhVSxEM6uJY3gBW/gu9Wk9i/t2qpse6ifuNT/31thiDe
X7vZ+UiAcM6v2OFT6o1So2smTgc8DovHCybBrAtg96VMFrsMHUQn3QKHkCxPCOLa
60PRNg8p9wKBgQCdE4CZh4OcMR+JK1lH9HeGdO/ITu7xROuivD80nEDHXDrgvzG9
KLlAp2qSO9+Ozjho3sFLoeMrV/T61dA8lMZDl1wMDALli4s4vpwMyBcwaSY1YCQE
GwmwindbE7xkckF42+gBsMwYG+F5DPsoWOm4O1ilTgPrXkxipoK68y4kSQKBgGvi
k8UQqNyys/v5g87bEdqG7mqC0R/ejW0kP1DlKHBZlInz7z2EgSfk+0e2AikfbiXH
cUyk/hY0Ke0/GW5n3Q+ZY2Oeed4YvRWJ3r8iZPRIgoDBEccVa/f0lthkVvsYzcsO
uydc5E7415Ly4UVCgz1rL6auomOAO08ZGOqxhvfxAoGBAJpAI+blOCPVAtU6KNxa
fLsSHnPS9yZ5cWg69MZ7C08zU8AkzT/FEvamPQy+yi5uPpxZPiI6/2P8xyx7AfsS
iZ4r94SPG2dIgnAoBwXLCMuajOJKACEBilGiSPBA8sLSfEZD38NKop05sFWLB2Wm
T336DWGmVQqjI5SOfxvmth67
-----END PRIVATE KEY-----
EOT
  certificate = <<EOT
-----BEGIN CERTIFICATE-----
MIIDVzCCAj8CFHLjdUyujHGHwQAbfeRSOCDC9n42MA0GCSqGSIb3DQEBCwUAMGcx
CzAJBgNVBAYTAlVTMRAwDgYDVQQIDAdNYXpvdmlhMQ8wDQYDVQQHDAZXYXJzYXcx
FDASBgNVBAoMC1NlZW1zIENsb3VkMQ0wCwYDVQQLDARSb290MRAwDgYDVQQDDAdS
b290IENBMB4XDTIzMTAyMjE4MDQ1N1oXDTI1MTAyMTE4MDQ1N1owaTELMAkGA1UE
BhMCVVMxEDAOBgNVBAgMB01hem92aWExDzANBgNVBAcMBldhcnNhdzEUMBIGA1UE
CgwLU2VlbXMgQ2xvdWQxDTALBgNVBAsMBFJvb3QxEjAQBgNVBAMMCWplZXAucmVz
dDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJz5/O+cdTn7esXaaTXn
R/+sn0iukU5FWrnJFNcIyarzttolV2L2TWMhVRS0HbqhxmTTlag8iPQ1Zr8OeTXj
MoK+eNFo/Eho3iRWxQx+dFqOEjBSRJPNVTk3LufxoEMq2Jo8AsqCsXsnL/0CZqEM
KRs9s1atQGkYgvTecm+oauD7P6L2bAqLiczX5EH06vzPCaEEHMk4AOXNA2Gmyncr
pwSsM3ag7Xu1t0Co7LaVAz2NZejHLJnitP2EEwmAGlAUPWVblYnc50+7T63YWoEM
89oTthzCp4QO03RjEUY5NIpfom7+r9ke9ceySfFE+fewRZBWLulg57pZTMD0nK4P
vBsCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAAhMmbv2on6uN1xTRTUE667txFx7S
3AYFHSdmC8uwx0GMjbsEs9v4eGuZ8/V4qAISUUavLmkWExvDTsjClwOpiUrs1hAt
73KVnZZmpb1n60LS86mqzSRyQ6zPDLjb6Bvnsd31+FL7Ttn918UGjDCnltvC/2k8
1KV15sX4AdI8Ez1Z1WxRd0AtWlxeLrj2udVlHrioY8WhQHSNros5Se+7uCi/4ZZi
m8GsNQJMr6zFJoXnRNneGZbgfDqDN4oooL172RGV87cLXmGfhB2eDaX2IIxh1t5k
6MTHWvd/HpMdCHvp3LEJ8RwtwfcCRloIgTtjJsjUEqwCN5UAEq0ZgRNcuw==
-----END CERTIFICATE-----
EOT
}
````
