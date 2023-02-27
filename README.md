<div align="center">
  <img src="https://github.com/web-stek/RESTRAPI/blob/main/reststrapi_logo.svg" />
  <h3 align="center">RESTRAPI Documentation & Help</h3>

  <p align="center">
    Collections of GraphQL queries and mutations that power your Strapi app!
    <br />
    <a href="https://github.com/kevinadhiguna/strapi-graphql-documentation#-table-of-contents"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/web-stek/RESTRAPI/issues">Report Bug</a>
    ·
    <a href="https://github.com/web-stek/RESTRAPI/pulls">Request Feature</a>
  </p>
</div>
<br>

<!-- TABLE OF CONTENTS -->
## 📚 Table of Contents

1. [About RESTRAPI](#-about-restrapi)
2. [UpdateMe](#-UpdateMe)
4. [Contact](#-contact)


## 👤 UpdateMe (lie “isOwner”)

Erstelle im Strapi-Ordner /src/extensions/user-permissions eine JS-Datei strapi-server.js.

Folgender Inhalt in die Datei strapi-server.js einfügen:

```jsx
module.exports = (plugin) => {
plugin.controllers.user.updateMe = async (ctx) => {
if (!ctx.state.user || ![ctx.state.user.id](http://ctx.state.user.id/)) {
return (ctx.response.status = 401);
}
await strapi
.query("plugin::users-permissions.user")
.update({
where: { id: [ctx.state.user.id](http://ctx.state.user.id/) },
data: ctx.request.body
})
.then((res) => {
ctx.response.status = 200;
});
};
```

```jsx
plugin.routes["content-api"].routes.push({
method: "PUT",
path: "/user/me",
handler: "user.updateMe",
config: {
prefix: "",
polices: [],
},
});
return plugin;
};
```

## 🌐 Contact
Author : Paco Krummenacher @web-stek

See on Github : [https://github.com/web-stek/](https://github.com/web-stek/)