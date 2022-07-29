# kured

Get the yaml file to customize:

```bash
latest=$(curl -s https://api.github.com/repos/weaveworks/kured/releases | jq -r .[0].tag_name)
wget "https://github.com/weaveworks/kured/releases/download/$latest/kured-$latest-dockerhub.yaml"
```

Let's customize the following arguments, then apply the yaml:

```bash
Flags:
      --alert-filter-regexp regexp.Regexp   alert names to ignore when checking for active alerts
      --alert-firing-only bool              only consider firing alerts when checking for active alerts
      --blocking-pod-selector stringArray   label selector identifying pods whose presence should prevent reboots
      --drain-grace-period int              time in seconds given to each pod to terminate gracefully, if negative, the default value specified in the pod will be used (default: -1)
      --skip-wait-for-delete-timeout int    when seconds is greater than zero, skip waiting for the pods whose deletion timestamp is older than N seconds while draining a node (default: 0)
      --ds-name string                      name of daemonset on which to place lock (default "kured")
      --ds-namespace string                 namespace containing daemonset on which to place lock (default "kube-system")
      --end-time string                     schedule reboot only before this time of day (default "23:59:59")
      --force-reboot bool                   force a reboot even if the drain is still running (default: false)
      --drain-timeout duration              timeout after which the drain is aborted (default: 0, infinite time)
  -h, --help                                help for kured
      --lock-annotation string              annotation in which to record locking node (default "weave.works/kured-node-lock")
      --lock-release-delay duration         hold lock after reboot by this duration (default: 0, disabled)
      --lock-ttl duration                   expire lock annotation after this duration (default: 0, disabled)
      --message-template-uncordon string    message template used to notify about a node being successfully uncordoned (default "Node %s rebooted & uncordoned successfully!")
      --message-template-drain string       message template used to notify about a node being drained (default "Draining node %s")
      --message-template-reboot string      message template used to notify about a node being rebooted (default "Rebooting node %s")
      --notify-url                          url for reboot notifications (cannot use with --slack-hook-url flags)
      --period duration                     reboot check period (default 1h0m0s)
      --prefer-no-schedule-taint string     Taint name applied during pending node reboot (to prevent receiving additional pods from other rebooting nodes). Disabled by default. Set e.g. to "weave.works/kured-node-reboot" to enable tainting.
      --prometheus-url string               Prometheus instance to probe for active alerts
      --reboot-command string               command to run when a reboot is required by the sentinel (default "/sbin/systemctl reboot")
      --reboot-days strings                 schedule reboot on these days (default [su,mo,tu,we,th,fr,sa])
      --reboot-delay duration               add a delay after drain finishes but before the reboot command is issued (default 0, no time)
      --reboot-sentinel string              path to file whose existence signals need to reboot (default "/var/run/reboot-required")
      --reboot-sentinel-command string      command for which a successful run signals need to reboot (default ""). If non-empty, sentinel file will be ignored.
      --slack-channel string                slack channel for reboot notifications
      --slack-hook-url string               slack hook URL for reboot notifications [deprecated in favor of --notify-url]
      --slack-username string               slack username for reboot notifications (default "kured")
      --start-time string                   schedule reboot only after this time of day (default "0:00")
      --time-zone string                    use this timezone for schedule inputs (default "UTC")
      --log-format string                   log format specified as text or json, defaults to "text"
```

I will set the following arguments:

```bash
      --alert-filter-regexp=^RebootRequired$
      --end-time=17:00
      --notify-url=smtp://username:password@host:port/?fromAddress=fromAddress&toAddresses=recipient1
      --prometheus-url=http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local
      --reboot-days=mo,tu,we,th,fr
      --start-time=10:00
      --time-zone=Europe/Rome
```

Be aware to put your proper url to the prometheus service in the `--prometheus-url=` flag.

To check for prometheus alerts (not checking kured's alerts), reboot between 9 and 17 in weekdays, Europe/Rome timezone, send notifications by email.

```bash
kubectl apply -f "kured-$latest-dockerhub.yaml"
```
