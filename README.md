# jestersimpps.github.io

Personal blog built with Jekyll using the HPSTR theme.

## Adding a New Blog Post

1. Create a new markdown file in `_posts/` with the naming format:
   ```
   YYYY-MM-DD-your-post-slug.md
   ```

2. Add frontmatter at the top:
   ```yaml
   ---
   layout: post
   title: "Your Post Title"
   description: "A short description for SEO and previews"
   date: YYYY-MM-DD
   author: "Jester Simpps"
   categories: [Development]
   tags: [Tag1, Tag2, Tag3]
   image: /images/your-image.jpg
   ---
   ```

3. Write your content in markdown below the frontmatter

4. If you have a cover image, add it to `images/`

## Running Locally

```bash
export PATH="/usr/local/opt/ruby@3.3/bin:$PATH"
cd /Users/jovinkenroye/Sites/jestersimpps.github.io
bundle exec jekyll serve --port 4001
```

Then open http://127.0.0.1:4001/

## Deploying

Push to the `master` branch - GitHub Pages will automatically build and deploy.

```bash
git add .
git commit -m "add new post"
git push
```
