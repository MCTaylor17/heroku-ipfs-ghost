# Ghost Blogging on Heroku with IPFS

Ghost is a free, open, simple blogging platform. Visit the project's website at <http://ghost.org>, or read the docs on <http://support.ghost.org>.

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

## Ghost v2.x

This has been forked from the [Ghost 1.x on Heroku by cobyism](https://github.com/cobyism/ghost-on-heroku). The **Deploy button** should "just work" -- you'll need to add S3 credentials after the fact if you want file uploads to work.

- Edited the `package.json` to include the newest 2.x Ghost release and the newest Casper and S3 adapter modules
- Removed `package-lock.json` so that newest packages are used automatically

### Things you should know

After deployment,
- First, visit Ghost at `https://YOURAPPNAME.herokuapp.com/ghost` to set up your admin account
- The app may take a few minutes to come to life
- Your blog will be publicly accessible at `https://YOURAPPNAME.herokuapp.com`
- If you subsequently set up a [custom domain](https://devcenter.heroku.com/articles/custom-domains) for your blog, you’ll need to update your Ghost blog’s `PUBLIC_URL` environment variable accordingly, `heroku config:set PUBLIC_URL=https://www.example.com`

#### 🚫🔻 Do not scale-up beyond a single dyno

[Ghost does not support multiple processes.](https://docs.ghost.org/faq/clustering-sharding-multi-server/)

If your Ghost app needs to support substantial traffic, then use a CDN add-on:

  * [Fastly](https://elements.heroku.com/addons/fastly)
  * [Edge](https://elements.heroku.com/addons/edge).

#### Using with file uploads disabled

Heroku app filesystems [aren’t meant for permanent storage](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem), so file uploads are disabled by default when using this repository to deploy a Ghost blog to Heroku. If you’re using Ghost on Heroku with S3 file uploads disabled, you should leave all environment variables beginning with `S3_…` blank.

#### Configuring S3 file uploads

To configure S3 file storage, create an S3 bucket on Amazon AWS, and then specify the following details as environment variables on the Heroku deployment page (or add these environment variables to your app after deployment via the Heroku dashboard):

- `S3_ACCESS_KEY_ID` and `S3_ACCESS_SECRET_KEY`: **Required if using S3 uploads**. These fields are the AWS key/secret pair needed to authenticate with Amazon S3. You must have granted this keypair sufficient permissions on the S3 bucket in question in order for S3 uploads to work.

- `S3_BUCKET_NAME`: **Required if using S3 uploads**. This is the name you gave to your S3 bucket.

- `S3_BUCKET_REGION`: **Required if using S3 uploads**. Specify the region the bucket has been created in, using slug format (e.g. `us-east-1`, `eu-west-1`). A full list of S3 regions is [available here](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region).

- `S3_ASSET_HOST_URL`: Optional, even if using S3 uploads. Use this variable to specify the S3 bucket URL in virtual host style, path style or using a custom domain. You should also include a trailing slash (example `https://my.custom.domain/`).  See [this page](http://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html) for details.

Once your app is up and running with these variables in place, you should be able to upload images via the Ghost interface and they’ll be stored in Amazon S3. :sparkles:

##### Provisioning an S3 bucket using an add-on

If you’d prefer not to configure S3 manually, you can provision the [Bucketeer add-on](https://devcenter.heroku.com/articles/bucketeer)
to get an S3 bucket (Bucketeer starts at $5/mo).

To configure S3 via Bucketeer, leave all the S3 deployment fields blank and deploy your
Ghost blog. Once your blog is deployed, run the following commands from your terminal:

```bash
# Provision an Amazon S3 bucket
heroku addons:create bucketeer --app YOURAPPNAME

# Additionally, the bucket's region must be set to formulate correct URLs
# (Find the "Region" in your Bucketeer Add-on's web dashboard.)
heroku config:set S3_BUCKET_REGION=us-east-1 --app YOURAPPNAME
```

### How this works

This repository is a [Node.js](https://nodejs.org) web application that specifies [Ghost as a dependency](https://docs.ghost.org/v1.0.0/docs/using-ghost-as-an-npm-module), and makes a deploy button available.

  * Ghost and Casper theme versions are declared in the Node app's [`package.json`](package.json)
  * Scales across processor cores in larger dynos via [Node cluster API](https://nodejs.org/dist/latest-v6.x/docs/api/cluster.html)

## Updating source code

Optionally after deployment, to push Ghost upgrades or work with source code, clone this repo (or a fork) and connect it with the Heroku app:

```bash
git clone https://github.com/bmann/fission-ghost
cd fission-ghost

heroku git:remote -a YOURAPPNAME
heroku info
```

Then you can push commits to the Heroku app, triggering new deployments:

```bash
git add .
git commit -m "Important changes"
git push heroku master
```

Watch the app's server-side behavior to see errors and request traffic:

```bash
heroku logs -t
```

See more about [deploying to Heroku with git](https://devcenter.heroku.com/articles/git).

### Upgrading Ghost

On each deployment, the Heroku Node/npm build process will **auto-upgrade Ghost to the newest 2.x version**. To prevent this behavior, use npm 5+ (or yarn) to create a lockfile.

```bash
npm install
git add package-lock.json
git commit -m 'Lock dependencies'
git push heroku master
```

Now, future deployments will always use the same set of dependencies.

To update to newer versions:

```
npm update
git add package-lock.json
git commit -m 'Update dependencies'
git push heroku master
```

### Database migrations

Requires MySQL database, available through either of two add-ons:
  * [JawsDB](https://elements.heroku.com/addons/jawsdb) (deploy default)
  * [ClearDB](https://elements.heroku.com/addons/cleardb)

Newer versions of Ghost frequently require changes to the database. These changes are automated with a process called **database migrations**.

After upgrading Ghost, you may see errors logged like:

> DatabaseIsNotOkError: Migrations are missing. Please run knex-migrator migrate.

To resolve this error, run the pending migrations and restart to get the app back on-line:

```bash
heroku run knex-migrator migrate --mgpath node_modules/ghost
heroku restart
```

This can be automated by adding the following line to `Procfile`:

```
release: knex-migrator migrate --mgpath node_modules/ghost
```

## Problems?

If you have problems using your instance of Ghost, you should check the [official documentation](http://support.ghost.org/) or open an issue on [the official issue tracker](https://github.com/TryGhost/Ghost/issues).

If you discover an issue with the deployment process provided by *this repository*, then [open an issue here](https://github.com/fission-suite/heroku-ipfs-ghost).

## License

Released under the [MIT license](./LICENSE), just like the Ghost project itself.
