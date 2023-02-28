<div align="center">
  <img src="https://github.com/web-stek/RESTRAPI/blob/main/reststrapi_logo.png" width="250px" height="250px" />
  <h3 align="center">RESTRAPI Documentation & Help</h3>

  <p align="center">
    Collections of REST API and mutations that power your Strapi app!
    <br />
    <a href="https://github.com/web-stek/RESTRAPI#-table-of-contents"><strong>Explore the docs »</strong></a>
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

1. [About RESTRAPI](#about-restrapi)
3. [Authentication/Auth](#-authentication)
    - [Signup](#signup)
    - [Login](#login)
    - [Forgot Password](#forgot-password)
    - [Logout](#logout)
2. [GET, POST, PUT & DELETE](#get-post-put-delete-add-update-delete)
    - [FetchMe](#fetchme)
    - [UpdateMe](#-updateme)
    - [UploadOneImage](#-uploadoneimage)
    - [Populate](#populate)
4. [Contact](#-contact)

# About RESTRAPI
I have created this documentation, because I have hardly found any information myself or there is hardly any information about it in the Strapi documentation. 
* Strapi version: 4.6.1
* JS library: React + Axios

The name RESTRAPI is formed from the terms: Strapi, REST, API.

# 🔑 Authentication
## Signup
Create a JSX File like `Signin.jsx` in your app and add the following code to submit a signin-form (username/identifier and password; example.com is your Strapi-Path +/- port).
```jsx
// Signin.jsx

const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const input = {
      username: formData.get("username"),
      email: formData.get("email"),
      password: formData.get("password"),
    };
    if (!validator.isEmail(input.email)) {
      setAlert(
        "Invalid email address, please enter a valid email address.",
        "error"
      );
      return;
    }
    try {
      const response = await axios.post(
        'http://example.com//api/auth/local/register',
        input
      );
      if (response && response.data) {
        setAlert(
          "Registration successful. Go to your mailbox and confirm the email address with the link sent.",
          "success"
        );
        setUsername("");
        setEmail("");
        setPassword("");
      } else {
        setAlert("Registration failed", "error");
      }
    } catch (err) {
      setAlert(`Registration failed: ${err.message}`, "error");
    }
  };
```
## Login
Create a JSX File like `Login.jsx` in your app and add the following code to submit a login-form (username/identifier and password; example.com is your Strapi-Path +/- port). The stored JWT-token will be used for accessing "Me".
```jsx
// Login.jsx

import axios from "axios";

const handleSubmit = async (e) => {
  e.preventDefault();
  try {
    const input = {
      identifier: identifier,
      password: password,
    };
    const response = await axios.post(
      "http://example.com/api/auth/local",
      input
    );
    // Store JWT token in browser's local storage
    localStorage.setItem("jwt", response.data.jwt);
    setAlert("Login Successful, Redirecting...", "success");
    setTimeout(() => {
      navigate("/");
    }, 1000);
  } catch (err) {
    setAlert("Invalid email or password", "danger");
  }
};
```

## Logout
Create a JSX File like `Signout.jsx` in your app and add the following code to submit a signout-button/link. I daded `navigate` (`react-dom`) to go to a spec. page after removing the JWT-token.

```jsx
// src/api/controllers/user/Logout.jsx

const handleLogout = () => {
    localStorage.removeItem("jwt");
    navigate("/example");
  };
  ```
## Forgot-Password
Create a JS File `forgot-password.js` in the strapi folder `/src/api/user/controllers`.

Add the following content to the file:

```jsx
// src/api/controllers/user/forgot-password.js

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
if necessary `strapi/utils` must be installed.

```jsx
yarn add @strapi/utils
```
# GET, POST, PUT, DELETE (add, update, delete)

## 👤FetchMe
```jsx
// /src/extensions/user-permissions/FetchUserMe.jsx
  const headers = {
    Authorization: `Bearer ${localStorage.getItem("jwt")}`,
  };

  useEffect(() => {
    const fetchUser = async () => {
      try {
        const response = await axios.get(
          'http://example.com//api/users/me',
          { headers }
        );
        const user = response.data;
        const username = user?.username || "";
        const firstNme = user?. firstName || ""; // new added field "firstName" in Strapi
        // ...
        } catch (error) {
        console.error(error);
      }
    };
    fetchUser();
  }, []);
```
## 👤 UpdateMe
This code solves the problem that the user can only edit their own fields and not mistakenly other (`findOne`, `find`) user data (similar to "isOwner").

Create a JS file `strapi-server.js` in the strapi folder `/src/extensions/user-permissions`.

Add the following content to the file:

```jsx
// /src/extensions/user-permissions/forgot-password.js

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

## 🖼️ UploadOneImage
This code solves the problem that the user can only edit their own fields and not mistakenly other (`findOne`, `find`) user data (similar to "isOwner").

Create a JS file `strapi-server.js` in the strapi folder `/src/extensions/user-permissions`.

Add the following content to the file:

```jsx
//Image Uploader
  const handleFileChange = (event) => {
    const selectedFile = event.target.files[0]; // store the selected file in a variable

    if (selectedFile.size > 1024 * 1024) {
      // check the size of the selected file
      setAlert("The image must not be larger than 1 MB.", "danger");
      setShowAlert(true);
      setTimeout(() => {
        setShowAlert(false);
      }, 5000);
      return;
    }

    setFile(selectedFile); // set the selected file as the state variable
  };

  const handleUploadAvatar = async () => {
    try {
      const formData = new FormData();
      formData.append("files", file);
      formData.append("refId", userId);
      formData.append("ref", "plugin::users-permissions.user");
      formData.append("field", "Image");

      const config = {
        method: "post",
        url: 'http://example.com/api/upload',
        headers: {
          Authorization: `Bearer ${localStorage.getItem("jwt")}`,
          "Content-Type": "multipart/form-data",
        },
        data: formData,
      };

      const response = await axios(config);
      const fileId = response.data[0].id;

      const updatedUser = await axios.put(
        'http://example.com/api/user/me',
        { Avatar: fileId },
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("jwt")}`,
          },
        }
      );

      setAlert("Updated user-avatar", "success");
      setShowAlert(true);
      setTimeout(() => {
        setShowAlert(false);
        window.location.reload(false);
      }, 5000);
    } catch (error) {
      setAlert("Error uploading the image", "danger");
      setShowAlert(true);
      setTimeout(() => {
        setShowAlert(false);
      }, 5000);
    }
  };
```

## 🧮 Populate
To get access to for ex. the Me-Role (Strapi User Role of Me) or Me-Media (Strapi Media-field) you can add `..me?populate=role,Media` in the `axios.get('http://example.com//api/users/me'`

## 🌐 Contact
* Author : Paco Krummenacher [https://github.com/web-stek/](@web-stek)
* See on Github : [https://github.com/web-stek/](https://github.com/web-stek/)