# ShiftRight Labs Website

Official website for ShiftRight Labs, hosted on GitHub Pages.

## Live Site
- https://shiftrightlabs.github.io
- https://www.shiftrightlabs.com (after DNS configuration)

## Local Development

Simply open `index.html` in your browser to preview locally.

## Deployment

Any push to the `main` branch automatically deploys to GitHub Pages.

```bash
git add .
git commit -m "Update website"
git push origin main
```

## DNS Configuration

Configure your DNS provider with these records:

```
Type    Name    Value
CNAME   www     shiftrightlabs.github.io.
A       @       185.199.108.153
A       @       185.199.109.153
A       @       185.199.110.153
A       @       185.199.111.153
```

## Custom Domain

The `CNAME` file contains `www.shiftrightlabs.com` which tells GitHub Pages to serve the site on your custom domain.
