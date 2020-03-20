# flyss

Flyss is a proof-of-concept for deploying a shadowsocks proxy server at little to no cost.

Fly.io is a platform for running Docker containers close to end users. It deploys microVMs to the edge and scales according to the demand. In this case, we use the Docker image from [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev/tree/master/docker/alpine) to run a lightweight and cost-effective proxy server. It's capable of tunneling up to 85G/mo for free thanks to the monthly credit provided for free.

## Usage

### Setup

1. Go through the official [hands-on guide](https://fly.io/docs/hands-on/start/)  (section 1-4) to install the fly command line tool `flyctl` and login.

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
    - `flyctl secrets set ARGS="-v"` to enable verbose mode (optional but recommended)
    - You can set more secrets according to the official [Dockerfile](https://github.com/shadowsocks/shadowsocks-libev/blob/master/docker/alpine/Dockerfile)

3. Deploy your app `flyctl deploy` and you are done! Fly will build your app on their server. You don't have to have Docker installed.
