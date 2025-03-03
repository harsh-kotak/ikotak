---
title: Bash Commands
weight: 10
---

<!-- Variable -->
{{< note title="Variable" >}}

```bash
NAME="John"
echo $NAME
echo "$NAME"
echo "${NAME}
```

{{< /note >}}

<!-- Condition -->
{{< note title="Condition" >}}

```bash
if [[ -z "$string" ]]; then
  echo "String is empty"
elif [[ -n "$string" ]]; then
  echo "String is not empty"
fi
```

{{< /note >}}

<!-- Filter -->
{{< note title="Filter" >}}

```bash
# Filter comment and empty lines from a file
grep -v '^$\|^\s*\#' temp

# Filter certs
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}’ cacerts.pem
```

{{< /note >}}

<!-- Validate WSS -->
{{< note title="WSS" >}}

```bash
# Verify endpoint is using secure web sockets
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" -H "Host: ikotak.com" -H "Origin: https://ikotak.com" https://ikotak.com -k
```

{{< /note >}}
