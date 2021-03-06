## Expose container level metrics using paas-prometheus-exporter

[Cloud Foundry][] provides time-series data (metrics), for your PaaS apps.

Currently supported metrics are:

* CPU
* RAM
* disk usage data
* app crashes
* app requests
* app response times

### Set up the metrics exporter app

Before you setting up the metrics exporter app, you'll need a live [Cloud Foundry account][] assigned to the spaces you want to receive metrics on.

Your new account should be separate to your primary Cloud Foundry account and use the [`SpaceAuditor`][] role beause it can view app data without modifying it.

To set up the metrics exporter app:

1. Clone the [paas-prometheus-exporter][] GitHub repository.
2. [Push the metrics exporter app][] to Cloud Foundry (without starting the app) by running
 * `cf push -f manifest.yml --no-start <app-name>`
3. Set the following mandatory environment variables in the metrics exporter app using
  * `cf set-env <app-name> NAME VALUE`

	Use the `cf set-env` command for these mandatory variables, this will keep secure the secret information contained in them.

	|Name|Value|
	|:---|:---|
	|`API_ENDPOINT`|Use `https://api.cloud.service.gov.uk`|
	|`USERNAME`|Cloud Foundry user|
	|`PASSWORD`|Cloud Foundry password|

	You could also set environment variables by amending the manifest file for optional environment variables that do not contain secret information. Read [paas-metric-exporter][] GitHub repository for more information.

4. Run `cf start <app-name>` to start your app
5. Check you're generating Prometheus metrics at the metrics endpoint
    - `https://<app-name>.cloudapps.digital/metrics`
6. Bind your app to the Prometheus service
  - `cf bind-service <app-name> <service-instance-name>`

### IP safelist your app

IP safelist is a security feature often used for limiting and controlling access to an app to trusted users. It works by only allowing traffic through from a list of trusted IP addresses or IP ranges.

By using the `paas-ip-authentication-route-service` you will only allow traffic from GDS Prometheis and [GDS Office IPs][].

1. Register the IP safelist route service as a user-provided service in your PaaS space.

    `cf create-user-provided-service paas-ip-authentication-route-service -r https://paas-ip-authentication-route-service.cloudapps.digital`


2. Register the route service for routes you want to protect.

    `cf bind-route-service cloudapps.digital paas-ip-authentication-route-service --hostname <your paas app route>`

    For example, `app-to-protect.cloudapps.digital` would be:

    `cf bind-route-service cloudapps.digital paas-ip-authentication-route-service --hostname app-to-protect`

### Troubleshooting

If you're not receiving metrics, check the [logs][] for the metrics exporter app or contact us at [#re-prometheus-support Slack channel][].

### Further reading

The Service Manual as more information about [monitoring the status of your service][].

[Cloud Foundry account]: https://docs.cloud.service.gov.uk/get_started.html
[Cloud Foundry]: https://docs.cloudfoundry.org/concepts/overview.html
[logs]: https://reliability-engineering.cloudapps.digital/#logging
[monitoring the status of your service]: https://www.gov.uk/service-manual/technology/monitoring-the-status-of-your-service
[paas-prometheus-exporter]: https://github.com/alphagov/paas-prometheus-exporter
[#re-prometheus-support Slack channel]: https://gds.slack.com/messages/CAF5H4N4Q/
[`SpaceAuditor`]: https://docs.cloud.service.gov.uk/orgs_spaces_users.html#space-auditor
[GDS office IPs]: https://sites.google.com/a/digital.cabinet-office.gov.uk/gds-internal-it/news/aviationhouse-sourceipaddresses
[Push the metrics exporter app]: https://docs.cloud.service.gov.uk/deploying_apps.html
