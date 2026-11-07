# Neha's Portfolio

A personal portfolio website built with HTML, CSS and Bootstrap 5, hosted on AWS with an automated deployment pipeline.

**Live site:** https://d399uifpvb5lji.cloudfront.net

## About the site

A responsive three-page site:

| Page | What it shows |
|---|---|
| **Home** | Introduction and links to GitHub and LinkedIn |
| **My Works** | Projects, internship experience, licenses and certifications |
| **Resume** | Resume page with a downloadable PDF |

**Projects featured:** APInspector (REST API testing tool), DiscoverIndia (travel recommendation website), Pingong Game (3D ping pong game built in Unity).

## Tech stack

- **Frontend:** HTML5, CSS3, Bootstrap 5
- **Hosting:** Amazon S3 (private bucket) + Amazon CloudFront (CDN, HTTPS)
- **CI/CD:** GitHub Actions
- **Security:** IAM role with least-privilege permissions, OIDC authentication, Origin Access Control

## Architecture

```
GitHub (push to main)
        |
        v
GitHub Actions --(OIDC, no stored keys)--> AWS IAM role
        |
        v
  aws s3 sync docs/  -->  S3 bucket (private)
                              |
                              v  (Origin Access Control)
                         CloudFront (HTTPS)
                              |
                              v
                          Visitors
```

### How it works

1. The site files live in the `docs/` folder.
2. Pushing to `main` triggers the workflow in `.github/workflows/deploy.yml`.
3. GitHub Actions signs in to AWS using **OpenID Connect (OIDC)**, so no AWS access keys are stored in the repository.
4. The workflow syncs `docs/` to the S3 bucket and invalidates the CloudFront cache so visitors see the latest version.

## AWS setup

- **S3:** the bucket is private, with Block Public Access turned on. Nothing can be read directly from S3.
- **CloudFront:** serves the site over HTTPS, redirects HTTP to HTTPS, with `index.html` as the default root object.
- **Origin Access Control (OAC):** the bucket policy allows only this CloudFront distribution to read objects.
- **IAM role for GitHub Actions:** trusted only by this repository's `main` branch. It can only list, upload and delete objects in the one bucket and create invalidations for the one distribution.
- **Cost and account safety:** CloudFront Free plan, an AWS Budgets zero-spend alert, and MFA on the root account.

## Run locally

No build step is needed. Clone the repo and open the site in a browser:

```bash
git clone https://github.com/Nehariyaz/portfolio.git
cd portfolio/docs
```

Then open `index.html`.

## Project structure

```
portfolio/
├── .github/
│   └── workflows/
│       └── deploy.yml     
├── docs/
│   ├── index.html
│   ├── myworks.html
│   ├── resume.html
│   ├── styles.css
│   └── assets/            
└── README.md
```

## What I learned

- Hosting a static website on S3 behind CloudFront
- Securing an S3 origin with Origin Access Control
- Setting up keyless CI/CD from GitHub Actions to AWS using OIDC
- Writing least-privilege IAM policies and debugging `AccessDenied` errors
- Keeping costs under control with budgets and free-tier limits
