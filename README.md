# vercel-server and firebase deploy

## Firebase Deploy

```bash
firebase init
#select hosting
#select existing project
#proceed  ->
y
#directory  ->
dist
#single page ->
y
```

## Server Deployment steps

1. comment await commands outside api methods for solving gateway timeout error

```js
//comment following commands
await client.connect();
await client.db("admin").command({ ping: 1 });
```

2. create vercel.json file for configuring server

```json
{
  "version": 2,
  "builds": [
    {
      "src": "index.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "index.js",
      "methods": ["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"]
    }
  ]
}
```

3. Add Your production domains to your cors configuration

```js
//Must remove "/" from your production URL
app.use(
  cors({
    origin: [
      "http://localhost:5173",
      "https://career-portal-ph.web.app",
      "https://career-portal-ph.firebaseapp.com",
    ],
    credentials: true,
  })
);
```

4. Let's create a cookie options for both production and local server for vercel

```js
const cookieOptions = {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: process.env.NODE_ENV === "production" ? "none" : "strict",
};
//localhost:5000 and localhost:5173 are treated as same site.  so sameSite value must be strict in development server.  in production sameSite will be none
// in development server secure will false .  in production secure will be true
```

## now we can use this object for cookie option to modify cookies

```js
//creating Token
app.post("/jwt", logger, async (req, res) => {
  const user = req.body;
  console.log("user for token", user);
  const token = jwt.sign(user, process.env.ACCESS_TOKEN_SECRET);

  res.cookie("token", token, cookieOptions).send({ success: true });
});

//clearing Token
app.post("/logout", async (req, res) => {
  const user = req.body;
  console.log("logging out", user);
  res
    .clearCookie("token", { ...cookieOptions, maxAge: 0 })
    .send({ success: true });
});
```

5. Deploy to Vercel

```bash
vercel
vercel --prod
# After completed the deployment . click on inspect link and copy the production domain
# setup your environment variables in vercel
# check your public API
```

<img src="https://i.ibb.co.com/dgH40d3/Screenshot-3.jpg"/>

# Server Deployment on Vercel Done

---

## ✅ Netlify Deployment (for `dist/` folder with reload fix)

1. **Ensure `dist/` is your build output**

> If you're using Vite or changed config, your output might be `dist/` instead of `build/`. That’s okay!

2. **Fix React reload issue**

➤ Add a file in your `public/` folder:

📄 **`public/_redirects`**

```
/*    /index.html   200
```

3. **Deploy to Netlify**

* Visit [https://app.netlify.com/](https://app.netlify.com/)
* Create a new site → drag & drop your `dist/` folder
* OR link your GitHub repo and set:

  * **Build command**: `npm run build`
  * **Publish directory**: `dist`

✅ Your SPA routes (like `/books/:id`) will now work even after refresh!

---

## ✅ Surge Deployment (for `dist/` folder with reload fix)

1. **Install Surge (if not already)**

```bash
npm install -g surge
```

2. **Fix reload issue (SPA support)**

➤ Copy `index.html` to `200.html` inside `dist/`:

```bash
cp dist/index.html dist/200.html
```

Surge uses `200.html` to serve fallback routes.

3. **Deploy your `dist/` folder**

```bash
surge ./dist
```

4. Enter email, password, and choose a custom or auto-generated domain like:

```
your-app.surge.sh
```

---

### ✅ Summary

| Platform | Build Folder | Reload Fix Required | What to Add                     |
| -------- | ------------ | ------------------- | ------------------------------- |
| Netlify  | `dist/`      | ✅ Yes               | `public/_redirects`             |
| Surge    | `dist/`      | ✅ Yes               | `dist/200.html` (copy of index) |

---
