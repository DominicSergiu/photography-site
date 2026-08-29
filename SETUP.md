# Setup guide

Everything here happens in your web browser. No terminal, no commands, nothing
to install. About an hour, most of it waiting for things to finish.

You'll create two free accounts along the way: GitHub (stores the files) and
Netlify (puts them online).

---

## Step 1 — Create a GitHub account

Skip if you already have one.

1. Go to **https://github.com/signup**
2. Enter your email, pick a password and a username
3. Verify your email when the message arrives

That's it. GitHub is just where the files live — you won't need to understand
anything else about it.

---

## Step 2 — Create a place for the files

1. Go to **https://github.com/new**
2. **Repository name:** type `photography-site`
3. Leave everything else as it is
4. Do **not** tick "Add a README file"
5. Click the green **Create repository** button

You'll land on a mostly empty page with some instructions. Ignore all of it.

---

## Step 3 — Upload the site files

On that same page, look for a link that says **uploading an existing file**.
It's in the middle of the page, in a sentence that reads something like
"…or push an existing repository…". Click it.

If you can't find the link, go to:
`https://github.com/YOURUSERNAME/photography-site/upload/main`

Now:

1. Open the `photography-site` folder on your computer in Finder or Explorer
2. Select **everything inside it** — the `index.html` file and the `admin`,
   `content` and `images` folders. Not the outer folder itself, the contents.
3. Drag them onto the GitHub upload area in your browser
4. Wait for the file names to finish appearing
5. Scroll down, click the green **Commit changes** button

Use Chrome, Edge, or Firefox for this — Safari sometimes refuses to upload
folders. If your browser won't take the folders, upload `index.html` on its own
first, then repeat the upload for each folder separately.

You should now see your files listed on the repo page.

---

## Step 4 — Tell the CMS where your files are

1. In your repo, click the **admin** folder
2. Click **config.yml**
3. Click the **pencil icon** (top right of the file box) to edit it
4. Find the third line:

   ```
   repo: YOUR-GITHUB-USERNAME/YOUR-REPO-NAME
   ```

5. Change it to your username and repo name, for example:

   ```
   repo: domlucaciu/photography-site
   ```

   Just those two words with a slash between. No web address, no `.git`.

6. Scroll down and click **Commit changes**, then **Commit changes** again in
   the popup

---

## Step 5 — Put the site online

1. Go to **https://app.netlify.com/signup**
2. Click **Sign up with GitHub** and authorise it
3. On your Netlify dashboard, click **Add new site** → **Import an existing
   project**
4. Click **GitHub**, authorise again if asked
5. Find `photography-site` in the list and click it
6. On the settings screen:
   - **Build command:** leave empty
   - **Publish directory:** type a single full stop: `.`
7. Click **Deploy**

Wait a minute. Netlify gives you a web address like
`gentle-marmot-4a2b.netlify.app`. Click it — your site is live with the
placeholder photos.

---

## Step 6 — Make the CMS login work

This is the fiddliest step. It's what lets you sign in to add photos. Take it
slowly and it's fine.

**First, tell GitHub about your site:**

1. Go to **https://github.com/settings/developers**
2. Click **New OAuth App**
3. Fill in three fields:
   - **Application name:** `Photography CMS`
   - **Homepage URL:** your Netlify address, e.g.
     `https://gentle-marmot-4a2b.netlify.app`
   - **Authorization callback URL:** `https://api.netlify.com/auth/done`

   That last one is the same for everybody. Copy it exactly. Don't put your own
   address there.
4. Click **Register application**
5. On the next page you'll see a **Client ID**. Copy it somewhere.
6. Click **Generate a new client secret**. Copy that too — GitHub only shows it
   once.

**Then, tell Netlify about GitHub:**

1. Back in Netlify, open your site
2. In the left sidebar click **Site configuration**
3. Click **Access & identity**, scroll down to **OAuth**
4. Click **Install provider** → choose **GitHub**
5. Paste in the Client ID and Client Secret you just copied
6. Save

**Test it:** go to `your-netlify-address.netlify.app/admin`. Click "Sign in with
GitHub", approve it, and the CMS should open with your three series listed.

---

## Step 7 — Use your own domain

1. In Netlify: **Site configuration** → **Domain management** → **Add a domain**
2. Type your domain, e.g. `dominiclucaciu.com`, and follow the prompts
3. Netlify shows you some DNS records — usually one A record and one CNAME
4. Log in wherever you bought the domain (GoDaddy, Namecheap, Google Domains,
   whoever), find the DNS or Nameservers section, and add exactly what Netlify
   showed you
5. Wait. This can take ten minutes or a few hours

The padlock (HTTPS) appears on its own once the domain starts working. If
Netlify says the certificate is pending, leave it an hour.

---

## Step 8 — Add your photos

Go to **yourdomain.com/admin** and sign in.

1. Click **Series** in the sidebar, then **Street**
2. Scroll to **Photos**. Delete the placeholder entries using the bin icon on
   each one.
3. Click **Add Photo** and fill in:
   - **Photo** — click it, then upload from your computer
   - **Caption** — e.g. `Hackney`
   - **Show on homepage** — tick this for the shots you want on the front page
4. Repeat for each photo
5. Click **Publish** at the top right, then **Publish now**

Wait about a minute and reload your site. Your photos are live.

To change the order, drag the photo entries up and down in the list.

From now on this is the whole workflow: open `/admin`, upload, publish. You
never touch GitHub again.

---

## Adding a new series later

Two small jobs.

**In the CMS:** click **New Series** and fill in:
- **Series name:** `Landscapes`
- **URL slug:** `landscapes` — lowercase, no spaces
- **Order in navigation:** `4`

Add your photos, then Publish.

**In GitHub, once:** the site keeps a list of which series exist, and you need
to add the new one to it.

1. Go to your repo → **content** folder → **series.json**
2. Click the pencil icon
3. Add the new slug to the list, keeping the quotes and commas:

   ```json
   ["street", "travel", "portraits", "landscapes"]
   ```

4. Click **Commit changes**

The navigation link appears by itself a minute later.

---

## Photo export settings

- Long edge around 1600 pixels
- JPEG, quality 80
- Any shape — tall, wide, square. The grid handles it.

Anything much over 500KB per file will make the site feel sluggish.

---

## When something goes wrong

**`/admin` is blank or spins forever**
The `repo:` line in `admin/config.yml` is wrong. It must be exactly
`yourusername/photography-site` — no `https://`, no `.git`.

**Login bounces back or errors**
The callback URL in your GitHub OAuth app isn't right. It has to be
`https://api.netlify.com/auth/done`, character for character.

**Published, but the site looks unchanged**
In Netlify, click the **Deploys** tab. If the top entry says "Failed", open it
and read the message. If it says "Published", your browser is showing a cached
copy — hold Shift and click reload.

**A new series isn't in the menu**
Its slug is missing from `content/series.json`, or it's spelled differently
there than in the CMS.
