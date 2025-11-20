This is the webserver that hosts results for the CCDC nmap bot. 
# Prerequisites
Instructions for downloading and installing golang for linux can be found on the golang website here: https://go.dev/doc/install.

Once go is installed, the next step is to build the webserver. 
1. Use  `git clone` to copy the repos into `/opt`
2. Chown directory ownership to a non-root user
3. The application can then be compiled using `go build`

```shell
cd /opt
https://github.com/MNCCDC-RedTeam/nmap-dashboard-rt.git
sudo chown <user>:<user_group> -R nmap-dashboard-rt/
cd nmap-dashboard-rt
go build
```

At this point there should be an executable in the project folder. 
# Webserver
The web application is looking to load information from `.env`. The only mandatory parameter is `API_BASE_URL` which should be set to the webserver URL. However an optional `PORT` parameter may be used as well. 

The `.env` configuration file should look like:
```.env
API_BASE_URL=http://<sub>.<domain>.<tld>
PORT=<port_number>
```

Next up, disabling debug mode for gin, a component of the webserver. This can be done using an environment variable.
```shell
export GIN_MODE=release
```

At this point, the webserver can be started. Upon first boot, the webserver will perform the following actions:
- Create a sqlite database called `dashboard.db`.
- Provision an `admin` user for the application.
- Give the `admin` user a randomly generated password, which will output to the terminal.

Use the admin user to log into the webserver to ensure it works properly. Once logged in, log out and create a new account. This account will be used for the scanner later in setup. After entering credentials and seeing a message stating a successful registration, log back into the admin account.

Next click Users in the navigation tab > find the new user > check Active and Scanner > Click Update User.

Next, Certbot will be used to enable https on the endpoint. The webserver in this example is hosted behind an nginx reverse proxy, which certbot should be able to handle without issue.
```shell
sudo certbot run -d <sub>.<domain>.<tld>
```

After getting HTTPS functional, update the `.env` file for the new URL. 
```
API_BASE_URL=https://<sub>.<domain>.<tld>
PORT=<port_number>
```

Most of the management functions of the application are handled with the Swagger UI. The Swagger UI is hosted at `https://<sub>.<domain>.<tld>/swagger/index.html`.  Next up is creating teams using the POST verb at `/teams`. By default you will get a JSON string that looks like this:
```json
{
  "hosts": [
    {}
  ],
  "iprange": "string",
  "name": "string",
  "tid": "string"
}
```

The JSON object is the same for the GET and POST verbs, so there are some unused fields in the POST request. The 2 fields that matter are `iprange` and `name`. A sample team creation request looks like so:
```json
{
  "hosts": [
    {}
  ],
  "iprange": "192.168.1.0/24",
  "name": "Test1",
  "tid": "string"
}
```

Any nmap-compatible host definition should work in the `iprange` field, so if you want to only hit certain hosts, it can be narrowed down
```json
{
  "hosts": [
    {}
  ],
  "iprange": "192.168.1.1-5",
  "name": "Test2",
  "tid": "string"
}
```

Next, run the GET verb on `/teams` to verify the teams were created successfully.
```json
[
  {
    "Name": "Test1",
    "IPRange": "192.168.1.0/24",
    "TID": "b2074aed-9642-47c1-b8ef-ad1e096f24b4",
    "Hosts": null
  },
  {
    "Name": "Test2",
    "IPRange": "192.168.1.1-5",
    "TID": "4ff272e0-370a-4f68-b79a-a99418370b6e",
    "Hosts": null
  }
]
```

Once these teams are created, they should be visible at `https://<sub>.<doimain>.<tld>/main.html`, assuming the session is still logged in.

NOTE: Delete doesn't appear to be functional at this time. 
