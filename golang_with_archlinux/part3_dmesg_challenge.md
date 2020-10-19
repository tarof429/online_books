# Part 3: Challenge: A function to parse dmesg

This code snippet parses lines from dmesg and displays lines where src and dst fields are not the same.

{% code title="main.go" %}
```go
import (
	"fmt"
	"regexp"
	"strings"
)

type Dmesg struct {
	timestamp string
	in        string
	out       string
	src       string
	dst       string
}

func nonLoopBackDmsg(lines []string) []Dmesg {
	var ret []Dmesg

	getField := func(line string, index int) (string, string) {

		fields := strings.Split(line, " ")[index]

		key := strings.Split(fields, "=")[0]
		value := strings.Split(fields, "=")[1]
		return key, value

	}

	for _, line := range lines {
		t := regexp.MustCompile(`\[UFW AUDIT\]|\[UFW ALLOW\]`)
		fields := t.Split(line, -1)

		tstamp := strings.Trim(fields[0], " ")
		body := strings.Trim(fields[1], " ")

		_, inField := getField(body, 0)
		_, outField := getField(body, 1)
		_, srcField := getField(body, 2)
		_, dstField := getField(body, 3)

		d := Dmesg{
			timestamp: tstamp,
			in:        inField,
			out:       outField,
			src:       srcField,
			dst:       dstField,
		}

		if d.src != d.dst {
			ret = append(ret, d)
		}

	}
	return ret
}

func main() {

	lines := []string{
		"[ 1552.201877] [UFW AUDIT] IN= OUT=lo SRC=127.0.0.1 DST=127.0.0.1 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=63084 DF PROTO=TCP SPT=58820 DPT=22 WINDOW=65495 RES=0x00 SYN URGP=0",
		"[ 6868.484303] [UFW ALLOW] IN= OUT=enp37s0 SRC=192.168.10.102 DST=192.168.10.1 LEN=94 TOS=0x00 PREC=0x00 TTL=64 ID=58159 DF PROTO=UDP SPT=49010 DPT=53 LEN=74",
		"[ 1552.201877] [UFW AUDIT] IN= OUT=lo SRC=127.0.0.1 DST=127.0.0.1 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=63084 DF PROTO=TCP SPT=58820 DPT=22 WINDOW=65495 RES=0x00 SYN URGP=0",
		"[ 6026.147651] [UFW ALLOW] IN= OUT=enp37s0 SRC=192.168.10.102 DST=64.4.54.254 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=45070 DF PROTO=TCP SPT=38044 DPT=443 WINDOW=64240 RES=0x00 SYN URGP=0",
		"[ 7032.103924] [UFW ALLOW] IN= OUT=enp37s0 SRC=192.168.10.102 DST=64.4.54.254 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=35703 DF PROTO=TCP SPT=38220 DPT=443 WINDOW=64240 RES=0x00 SYN URGP=0",
	}

	ret := nonLoopBackDmsg(lines)
	fmt.Println(ret)
}
```
{% encode %}

The result is:

```bash
$ go run main.go
{[ 6868.484303]  enp37s0 192.168.10.102 192.168.10.1}
{[ 6026.147651]  enp37s0 192.168.10.102 64.4.54.254}
{[ 7032.103924]  enp37s0 192.168.10.102 64.4.54.254}
```