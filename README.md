# 欲說還休

Personal blog powered by [Hugo](https://gohugo.io/) + [hugo-ink](https://github.com/vinooganesh/hugo-ink) theme, deployed via GitHub Actions to [blog.terryx.com](https://blog.terryx.com/).

## Local Development

```bash
# Install Hugo (macOS)
brew install hugo

# Clone with submodules
git clone --recursive https://github.com/lele94218/lele94218.github.com.git
cd lele94218.github.com

# Start dev server
hugo server -D
```

Open http://localhost:1313 to preview.

## Writing a New Post

Create a new Markdown file in `content/posts/`:

```bash
hugo new posts/my-new-post.md
```

Or manually create the file with this frontmatter:

```markdown
---
title: "Post Title"
date: 2024-01-01
tags:
  - "Tag1"
  - "Tag2"
---

Content goes here...
```

### Math Support

Add `math: true` to frontmatter to enable KaTeX:

```markdown
---
title: "Math Post"
date: 2024-01-01
math: true
---

Inline: $E = mc^2$

Display:
$$\int_0^1 f(x) \, dx$$
```

### Code Blocks

Use fenced code blocks with language hints:

````markdown
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```
````

## Publishing

Push to `main` branch and GitHub Actions will automatically build and deploy:

```bash
git add .
git commit -m "Add new post"
git push origin main
```

Deployment typically takes 1-2 minutes. Check progress in the [Actions](https://github.com/lele94218/lele94218.github.com/actions) tab.

## Project Structure

```
content/
  posts/          # Blog posts
  about.md        # About page
assets/css/
  custom.css      # Custom styles
  chroma.css      # Syntax highlighting (light/dark)
layouts/          # Hugo layout overrides
static/images/    # Static assets (banner, etc.)
hugo.toml         # Site configuration
```
