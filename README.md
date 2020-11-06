## Pre-reqs

* docker
* docker-compose

## Purpose

This repo contains the files needed to demonstrate how to protect an api using Tyk Gateway open source edition.

_It is not production ready and should only be used for demonstrations._

# Tyk
We'll  start with getting Tyk running first, then add protection via keyclock.

## Spin-up tyk-redis

Tyk gateway uses Redis as a configuration store.  We use `docker-compose` to handle the communications between `tyk-gateway` and `redis`.

`docker-compose up -d tyk-redis`

## Spin-up Tyk

`docker-compose up -d tyk`

After starting tyk check that it running:
`docker-compose ps`

# Keycloak

The docker compose file contains definitions for keycloak and postgres.

## Spin-up keycloak-db

`docker-compose up -d keycloak-db`

Check that it's running:

`docker-compose ps`

## Spin-up keycloak

`docker-compose up -d keycloak`

## Configure keycloak

* In your browser go to `http://localhost:8180`
* Select the "Administrative Console" and  click "Clients" on the left-hand navigation.
* Click "Create" and on the "Add Client" page use 'tyk' as the Client ID, then click "Save".
* Click on the Details for the tyk client and enter 'confidential' for Access Type, http://localhost:8080 for Root URL and /mock/* for Valid Redirect URIs, then click "Save".
* On the Settings tab, ensure that the "Service Accounts Enabled" is set to on for this client.
* On the "Mappers" tab click "create" to add a new mapper.
* On the create screen, set the name to "tyk-audience-mapper", select "Audience" as the "Mapper Type", and choose tyk for the "Included Client Audience".

## Testing

Request a client token from keycloak:
`curl http://$HOST_IP:8180/auth/realms/master/protocol/openid-connect/token -d "grant_type=client_credentials" -d "client_id=tyk" -d "client_secret=<secret from client credentials>" -d "scope=openid"`

Copy the value of the `access_token` returned in the json object.

Request a protected resource:

`curl http://localhost:8080/mock/123 -H "Authorization: Bearer <access_token returned from previous curl>"`

eg.

```
curl http://localhost:8080/mock/123 -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJZNEdTUjE1TXEyUTBZMFp4bFkxVDZSYlhqTUhDV2Z2YXB0TUstZWY1VXRjIn0.eyJleHAiOjE2MDQ2NDU3MDksImlhdCI6MTYwNDY0NTY0OSwianRpIjoiMzVlNTEzNjEtZGFmOC00MmM3LTljYjEtYWRkZmZmYmRmN2FiIiwiaXNzIjoiaHR0cDovLzE5Mi4xNjguMS4xNTc6ODE4MC9hdXRoL3JlYWxtcy9tYXN0ZXIiLCJhdWQiOlsidHlrIiwiYWNjb3VudCJdLCJzdWIiOiJmZWY3ZmJhNi01YjA4LTRlZTAtOTJjNS02NDQ0ZDY5NzU3Y2EiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJ0eWsiLCJzZXNzaW9uX3N0YXRlIjoiMTkxNWM4N2ItOTcyOS00NTg3LWFjN2ItOGRjN2MwYTg5MTJjIiwiYWNyIjoiMSIsInJlYWxtX2FjY2VzcyI6eyJyb2xlcyI6WyJvZmZsaW5lX2FjY2VzcyIsInVtYV9hdXRob3JpemF0aW9uIl19LCJyZXNvdXJjZV9hY2Nlc3MiOnsiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIiwidmlldy1wcm9maWxlIl19fSwic2NvcGUiOiJvcGVuaWQgcHJvZmlsZSBlbWFpbCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwiY2xpZW50SWQiOiJ0eWsiLCJjbGllbnRIb3N0IjoiMTcyLjE5LjAuMSIsInByZWZlcnJlZF91c2VybmFtZSI6InNlcnZpY2UtYWNjb3VudC10eWsiLCJjbGllbnRBZGRyZXNzIjoiMTcyLjE5LjAuMSJ9.LDUk-nVtKny-9i525yuVZLH3WM42EPEHfHDeJhwq4UCV2csgBTDF2bLAeqBF4TdyLmaLQnVbcDhe6Bkvk5cEdcB4rdXtX490mo0KSW_IfgOfgsGISwccUx5oUbrs1GeJmZyBf9QYfmJ2h2_FpDsF8-q8-hNJKaxGipGokGjy1J6IGPhecDuHIrlWjXXSqjjmmuv0nJKcdzQBAb0AQKM_ym7ixljrwE6rzgBCWlu07YmDwoL9oUetLsUUabn4dwLW7zKRItbRQ3eED_byRs7D8yqNle0PTK2F76-1uWmmktdyWYvpd76nf_gxBtILHCHsjDFkC7mQBEz1cFRHpr084A"
```

If everything is configured correctly, you should see a response simiar to this:

```
{
  "startedDateTime": "2020-11-06T06:54:43.762Z",
  "clientIPAddress": "172.18.0.1",
  "method": "GET",
  "url": "http://mockbin.org/request/123",
  "httpVersion": "HTTP/1.1",
  "cookies": {},
  "headers": {
    "host": "mockbin.org",
    "connection": "close",
    "x-forwarded-for": "172.18.0.1, 172.17.0.2, 3.12.247.135",
    "x-forwarded-proto": "http",
    "x-forwarded-host": "mockbin.org",
    "x-forwarded-port": "80",
    "x-real-ip": "81.153.178.112",
    "user-agent": "curl/7.64.1",
    "accept": "*/*",
    "authorization": "Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJZNEdTUjE1TXEyUTBZMFp4bFkxVDZSYlhqTUhDV2Z2YXB0TUstZWY1VXRjIn0.eyJleHAiOjE2MDQ2NDU3MDksImlhdCI6MTYwNDY0NTY0OSwianRpIjoiMzVlNTEzNjEtZGFmOC00MmM3LTljYjEtYWRkZmZmYmRmN2FiIiwiaXNzIjoiaHR0cDovLzE5Mi4xNjguMS4xNTc6ODE4MC9hdXRoL3JlYWxtcy9tYXN0ZXIiLCJhdWQiOlsidHlrIiwiYWNjb3VudCJdLCJzdWIiOiJmZWY3ZmJhNi01YjA4LTRlZTAtOTJjNS02NDQ0ZDY5NzU3Y2EiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJ0eWsiLCJzZXNzaW9uX3N0YXRlIjoiMTkxNWM4N2ItOTcyOS00NTg3LWFjN2ItOGRjN2MwYTg5MTJjIiwiYWNyIjoiMSIsInJlYWxtX2FjY2VzcyI6eyJyb2xlcyI6WyJvZmZsaW5lX2FjY2VzcyIsInVtYV9hdXRob3JpemF0aW9uIl19LCJyZXNvdXJjZV9hY2Nlc3MiOnsiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIiwidmlldy1wcm9maWxlIl19fSwic2NvcGUiOiJvcGVuaWQgcHJvZmlsZSBlbWFpbCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwiY2xpZW50SWQiOiJ0eWsiLCJjbGllbnRIb3N0IjoiMTcyLjE5LjAuMSIsInByZWZlcnJlZF91c2VybmFtZSI6InNlcnZpY2UtYWNjb3VudC10eWsiLCJjbGllbnRBZGRyZXNzIjoiMTcyLjE5LjAuMSJ9.LDUk-nVtKny-9i525yuVZLH3WM42EPEHfHDeJhwq4UCV2csgBTDF2bLAeqBF4TdyLmaLQnVbcDhe6Bkvk5cEdcB4rdXtX490mo0KSW_IfgOfgsGISwccUx5oUbrs1GeJmZyBf9QYfmJ2h2_FpDsF8-q8-hNJKaxGipGokGjy1J6IGPhecDuHIrlWjXXSqjjmmuv0nJKcdzQBAb0AQKM_ym7ixljrwE6rzgBCWlu07YmDwoL9oUetLsUUabn4dwLW7zKRItbRQ3eED_byRs7D8yqNle0PTK2F76-1uWmmktdyWYvpd76nf_gxBtILHCHsjDFkC7mQBEz1cFRHpr084A",
    "accept-encoding": "gzip",
    "x-request-id": "51c5f679-8b06-4db6-b0f6-a07dbd421161",
    "via": "1.1 vegur",
    "connect-time": "0",
    "x-request-start": "1604645683755",
    "total-route-time": "0"
  },
  "queryString": {},
  "postData": {
    "mimeType": "application/octet-stream",
    "text": "",
    "params": []
  },
  "headersSize": 1753,
  "bodySize": 0
}
```
