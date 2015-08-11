The 42 API provide programmatic access to read and write 42's data. You can get and interact with the whole intranet data, and do things such creating a new message on the forum, read users profiles, get datas on a project, and more.
On his second version, it identifies 42 applications and users using OAuth, and responses are available in JSON.

Whether you're looking to build an official 42 integration for your service, or you just want to build something awesome, [we can help you get started](#getting_started).

Specifications
--------------

The API works over the https protocol. and accessed from the *api.intrav2.42.fr* domain.
- The current endpoint is https://api.intrav2.42.fr/v2.
- All data is sent and received as JSON.
- Blank fields are included as null instead of being omitted.
- All timestamps are returned in ISO 8601 format
