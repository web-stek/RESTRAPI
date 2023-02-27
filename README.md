<div align="center">
  <img src="https://github.com/web-stek/RESTRAPI/blob/main/reststrapi_logo.png" width="250px" height="250px" />
  <h3 align="center">RESTRAPI Documentation & Help</h3>

  <p align="center">
    Collections of REST API and mutations that power your Strapi (v4.6.1) app!
    <br />
    <a href="https://github.com/web-stek/RESTRAPI"><strong>Explore the docs »</strong></a>
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
3. [Authentication/Auth](#-Authentication)
    - [Forgot Password](#-forgot-password)
    - [Me Query](#-me-query)
4. [Contact](#-contact)


## 👤 UpdateMe
This code solves the problem that the user can only edit their own fields and not mistakenly other (findOne, find) user data (like "isOwner").

Create a JS file strapi-server.js in the strapi folder /src/extensions/user-permissions.

Add the following content to the file:

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

## 🔑 forgot-Password
Create a JS File forgot-password.js in the strapi folder /src/api/user/controllers.

Add the following content to the file:

```jsx
// api/controllers/forgot-password.js
const { sanitizeEntity } = require("@strapi/utils");
const { get } = require("lodash");

module.exports = {
  async forgotPassword(ctx) {
    const { email } = ctx.request.body;

    // Find the user with the given email address
    const [user] = await strapi.query("user", "users-permissions").find({
      email,
    });

    if (!user) {
      return ctx.throw(404, "User not found.");
    }

    const token = await strapi.plugins[
      "users-permissions"
    ].services.jwt.issueResetPasswordToken(user);

    const siteUrl = get(strapi, "config.server.siteUrl", "http://localhost");
    const resetUrl = `${siteUrl}/reset-password?code=${token}`;

    try {
      await strapi.plugins["email"].services.email.send({
        to: email,
        subject: "Password reset request",
        html: `
          <p>You recently requested to reset your password for your account. Click the button below to reset it.</p>
          <a href="${resetUrl}" target="_blank" style="display: block; width: 200px; height: 40px; background-color: #3f51b5; color: #fff; text-align: center; line-height: 40px; text-decoration: none; border-radius: 4px; margin: 10px auto;">Reset password</a>
          <p>If you did not request a password reset, please ignore this email.</p>
        `,
      });
    } catch (err) {
      ctx.throw(500, "Error sending password reset email.");
    }

    return sanitizeEntity(user, {
      model: strapi.query("user", "users-permissions").model,
    });
  },
};
```
if necessary strapi/utils must be installed.

```jsx
yarn add @strapi/utils
```

## 🌐 Contact
Author : Paco Krummenacher [https://github.com/web-stek/](@web-stek)
See on Github : [https://github.com/web-stek/](https://github.com/web-stek/)