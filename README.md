# Flyss

Flyss is a proof-of-concept for deploying a shadowsocks proxy server at little to no cost.

Fly.io is a platform for running Docker containers close to end users. It deploys microVMs to the edge and scales according to the demand. In this case, we use the Docker image from shadowsocks-libev to run a lightweight and cost-effective proxy server. It's capable of tunneling up to 85G/mo, thanks to the monthly credit provided for free.

## Usage

### Setup

1. Go through the section 1 to 4 of the official hands-on guide to install the fly command line tool flyctl and login.

```bash
$ flyctl auth signup
$ flyctl auth login
Opening browser to url https://fly.io/app/auth/cli/XXX
Waiting for session...Done
Successfully logged in as XXX@XXX.com
```

2. Clone this repo and cd into it

```bash
$ git clone https://github.com/davidzyx/flyss.git
$ cd flyss
```

### Configure

1. Create a fly app. Follow the on-screen prompt. This will generate a `fly.toml` config file.

```bash
$ flyctl apps create
```

2. Modify the content of `fly.toml` according to `fly-template.toml` using a text editor, ignoring the part specifying the app name. Or you can run the commands below.

```bash
$ { head -1 fly.toml; tail -n +2 fly-template.toml; } > temp
$ mv temp fly.toml
```

### Deploy

1. Check which region you want to deploy your proxy server to. e.g. the code for Los Angeles is `lax`. This way

```pre
$ flyctl platform regions
  CODE   NAME                          
  ams    Amsterdam, Netherlands        
  atl    Atlanta, Georgia (US)         
  dfw    Dallas 2, Texas (US)          
  ewr    Parsippany, NJ (US)           
  fra    Frankfurt, Germany            
  iad    Ashburn, Virginia (US)        
  lax    Los Angeles, California (US)  
  mrs    Marseille, France             
  nrt    Tokyo, Japan                  
  ord    Chicago, Illinois (US)        
  sea    Seattle, Washington (US)      
  sin    Singapore                     
  sjc    Sunnyvale, California (US)    
  syd    Sydney, Australia             
  yyz    Toronto, Canada 

$ flyctl scale regions lax=1
```

2. Set password and more arguments
    - `flyctl secrets set PASSWORD=<pass-here>` to set the ss-server password
    - `flyctl secrets set METHOD=<method>` to set encryption method, default it `aes-256-gcm`
    <!-- - `flyctl secrets set ARGS="-v"` to enable verbose mode (optional but recommended) -->
    - You can set more secrets according to the official [Dockerfile](https://github.com/shadowsocks/shadowsocks-libev/blob/master/docker/alpine/Dockerfile)

3. Deploy your app flyctl deploy and you are done! Fly will build it on their server. You don't have to have Docker installed. You should see the following messages.

```
$ flyctl deploy
...
v0 is being deployed
1 desired, 1 placed, 0 healthy, 0 unhealthy [health checks: 1 total]
v0 deployed successfully
```

### Connect and Monitor

1. Do `flyctl info` and you should see the server's hostname and ip.

```$ flyctl info
App
  Name     = XXXXX-XXX-XXXX          
  Owner    = user                    
  Version  = 0                            
  Status   = running                      
  Hostname = XXXXX-XXX-XXXX.fly.dev  

Services
  PROTOCOL   PORTS            
  TCP        5000 => 8388 []  

IP Addresses
  TYPE   ADDRESS                             CREATED AT  
  v4     XX.XX.XXX.XXX                       1m56s ago   
  v6     XXXX:XXXX:XXXX::                    1m55s ago   

$ flyctl info
...
Allocations
  ID         VERSION   REGION   DESIRED   STATUS    HEALTH CHECKS   CREATED     
  4e051761   0         lax      run       running   1 passing       2m6s ago  
```

2. Download a shadowsocks client from https://shadowsocks.org/en/download/clients.html and connect using the following credentials.

```json
{
    "server": "XXXXX-XXX-XXXX.fly.dev",
    "server_port": 5000,
    "local_address": "0.0.0.0",
    "local_port": 1080,
    "password": "your_pass_here",
    "timeout": 600,
    "method": "aes-256-gcm"
}
```

3. (Optional) Monitor all logs from the container by

```
$ flyctl logs
2020-03-20T21:36:23.848Z 4e051761 lax [info] [2020-03-20T21:36:23Z] Starting init...
2020-03-20T21:36:23.871Z 4e051761 lax [info] [2020-03-20T21:36:23Z] Running: /bin/sh -c exec ss-server       -s $SERVER_ADDR       -p $SERVER_PORT       -k ${PASSWORD:-$(hostname)}       -m $METHOD       -t $TIMEOUT       -d $DNS_ADDRS       -u       -v       $ARGS as nobody
2020-03-20T21:36:23.880Z 4e051761 lax [info] [01;32m 2020-03-20 21:36:23 INFO: UDP relay enabled
2020-03-20T21:36:23.886Z 4e051761 lax [info] [01;32m 2020-03-20 21:36:23 INFO: This system doesn't provide enough entropy to quickly generate high-quality random numbers.
2020-03-20T21:36:23.886Z 4e051761 lax [info] [01;32m 2020-03-20 21:36:23 INFO: initializing ciphers... aes-256-gcm
2020-03-20T21:36:23.890Z 4e051761 lax [info] Installing the rng-utils/rng-tools, jitterentropy or haveged packages may help.
2020-03-20T21:36:23.891Z 4e051761 lax [info] On virtualized Linux environments, also consider using virtio-rng.
2020-03-20T21:36:23.894Z 4e051761 lax [info] The service will not start until enough entropy has been collected.
2020-03-20T21:36:23.895Z 4e051761 lax [info] {"app": XXXX, "alloc": "4e051761", "region": "lax", "message": "\r", "stream": "stdout"}
2020-03-20T21:36:23.902Z 4e051761 lax [info] [01;32m 2020-03-20 21:36:23 INFO: using nameserver: 8.8.8.8,8.8.4.4
2020-03-20T21:36:23.903Z 4e051761 lax [info] [01;32m 2020-03-20 21:36:23 INFO: tcp server listening at 0.0.0.0:8388
2020-03-20T21:36:23.903Z 4e051761 lax [info] [01;32m 2020-03-20 21:36:23 INFO: udp server listening at 0.0.0.0:8388
2020-03-20T21:37:19.153Z 4e051761 lax [info] [01;32m 2020-03-20 21:37:19 INFO: new connection from client, 1 opened client connections
2020-03-20T21:37:19.154Z 4e051761 lax [info] [01;32m 2020-03-20 21:37:19 INFO: close a connection from client, 0 opened client connections
2020-03-20T21:37:19.317Z 4e051761 lax [warning] Health check status changed to 'passing' => TCP connect 172.18.9.XXX:8388: Success
...
```
