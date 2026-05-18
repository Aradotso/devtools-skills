---
name: launchkit-devtool-landing-template
description: Free HTML landing page template for developer tools and open source products by Evil Martians
triggers:
  - create a landing page for my dev tool
  - set up LaunchKit template
  - customize LaunchKit theme colors
  - build a developer tool landing page
  - use the Evil Martians landing template
  - create an open source product landing page
  - deploy LaunchKit HTML template
  - modify LaunchKit color scheme
---

# LaunchKit Dev Tool Landing Template

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

LaunchKit is a free, ready-to-deploy HTML landing page template specifically designed for developer tools and open source products. It's a static HTML/CSS/JS template that requires no build process and can be deployed immediately to any static hosting service.

## Installation

Clone the repository:

```bash
git clone https://github.com/evilmartians/devtool-template.git my-landing-page
cd my-landing-page
```

Or use it as a GitHub template by clicking "Use this template" on the repository page.

## Project Structure

```
├── index.html          # Main landing page
├── index.css           # Styles and CSS variables
├── index.js            # Interactive components
├── images/             # Image assets
└── fonts/              # Custom fonts
```

## Quick Start

1. **Choose your blocks**: Edit `index.html` and remove sections you don't need
2. **Replace content**: Update all placeholder text with your product copy
3. **Add images**: Upload your images to the `images/` folder
4. **Configure buttons**: Update href attributes and add tracking
5. **Deploy**: Upload to any static hosting

## Theme Customization

### Light/Dark Mode

Add the `theme-dark` class to the `<html>` element for dark theme:

```html
<!-- Light theme (default) -->
<html lang="en">

<!-- Dark theme -->
<html lang="en" class="theme-dark">
```

### Color Palette Customization

Edit the CSS variables in `index.css`:

```css
:root {
  /* Primary brand color */
  --color-primary-h: 210;
  --color-primary-s: 100%;
  --color-primary-l: 50%;
  
  /* Accent color */
  --color-accent-h: 350;
  --color-accent-s: 100%;
  --color-accent-l: 60%;
  
  /* Background */
  --color-bg: #ffffff;
  --color-bg-secondary: #f5f5f5;
  
  /* Text */
  --color-text: #1a1a1a;
  --color-text-secondary: #666666;
}

.theme-dark {
  --color-bg: #1a1a1a;
  --color-bg-secondary: #2a2a2a;
  --color-text: #ffffff;
  --color-text-secondary: #cccccc;
}
```

Use the [color theme tool](https://codepen.io/romanshamin/full/QwWgNLN) to generate matching light/dark palettes.

## Common HTML Patterns

### Hero Section

```html
<section class="hero">
  <div class="container">
    <h1 class="hero__title">Your Dev Tool Name</h1>
    <p class="hero__description">
      A brief, compelling description of what your tool does
    </p>
    <div class="hero__cta">
      <a href="#get-started" class="button button--primary">Get Started</a>
      <a href="https://github.com/yourname/repo" class="button button--secondary">
        View on GitHub
      </a>
    </div>
  </div>
</section>
```

### Features Grid

```html
<section class="features">
  <div class="container">
    <h2 class="section__title">Key Features</h2>
    <div class="features__grid">
      <div class="feature">
        <div class="feature__icon">🚀</div>
        <h3 class="feature__title">Fast Setup</h3>
        <p class="feature__description">Get started in minutes</p>
      </div>
      <div class="feature">
        <div class="feature__icon">🔧</div>
        <h3 class="feature__title">Easy Configuration</h3>
        <p class="feature__description">Minimal config required</p>
      </div>
    </div>
  </div>
</section>
```

### Code Example Block

```html
<section class="code-example">
  <div class="container">
    <h2 class="section__title">Quick Start</h2>
    <pre class="code-block"><code>npm install your-tool
npx your-tool init
your-tool run</code></pre>
  </div>
</section>
```

### CTA Section

```html
<section class="cta">
  <div class="container">
    <h2 class="cta__title">Ready to get started?</h2>
    <p class="cta__description">Join thousands of developers using this tool</p>
    <a href="#signup" class="button button--large button--primary">
      Start Free Trial
    </a>
  </div>
</section>
```

## JavaScript Interactivity

Add interactive elements in `index.js`:

```javascript
// Mobile menu toggle
const menuButton = document.querySelector('.menu-toggle');
const nav = document.querySelector('.nav');

menuButton?.addEventListener('click', () => {
  nav.classList.toggle('nav--open');
});

// Smooth scroll to anchor links
document.querySelectorAll('a[href^="#"]').forEach(anchor => {
  anchor.addEventListener('click', function (e) {
    e.preventDefault();
    const target = document.querySelector(this.getAttribute('href'));
    target?.scrollIntoView({ behavior: 'smooth' });
  });
});

// Track CTA clicks
document.querySelectorAll('.button--primary').forEach(button => {
  button.addEventListener('click', () => {
    // Analytics tracking
    if (window.gtag) {
      gtag('event', 'cta_click', {
        'button_text': button.textContent
      });
    }
  });
});
```

## Adding Analytics

Add Google Analytics or Plausible to `index.html`:

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>

<!-- Or Plausible -->
<script defer data-domain="yourdomain.com" src="https://plausible.io/js/script.js"></script>
```

Replace `GA_MEASUREMENT_ID` with your actual ID (use environment variables in your build process).

## Deployment

### GitHub Pages

1. Push your code to GitHub
2. Go to Settings → Pages
3. Select branch (usually `main`) and `/` (root)
4. Your site will be live at `https://username.github.io/repo-name`

### Netlify

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Deploy
netlify deploy --prod
```

Or connect your GitHub repo in the Netlify dashboard.

### Vercel

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel --prod
```

### Custom Server

Upload all files to your web server's public directory via FTP/SFTP:

```bash
# Using rsync
rsync -avz --exclude '.git' ./ user@yourserver.com:/var/www/html/
```

## Responsive Design

The template is mobile-first. Breakpoints are defined in `index.css`:

```css
/* Mobile: default styles */
.container {
  padding: 0 16px;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: 0 32px;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

## Image Optimization

Optimize images before uploading:

```bash
# Using ImageOptim CLI (macOS)
imageoptim images/**/*.{jpg,png}

# Using imagemin
npx imagemin images/*.{jpg,png} --out-dir=images/optimized
```

Use WebP format for better compression:

```html
<picture>
  <source srcset="images/hero.webp" type="image/webp">
  <img src="images/hero.jpg" alt="Hero image">
</picture>
```

## SEO Configuration

Update meta tags in `index.html`:

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Dev Tool - Tagline</title>
  <meta name="description" content="Your concise product description for search results">
  
  <!-- Open Graph -->
  <meta property="og:title" content="Your Dev Tool">
  <meta property="og:description" content="Your product description">
  <meta property="og:image" content="https://yourdomain.com/images/og-image.jpg">
  <meta property="og:url" content="https://yourdomain.com">
  
  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Your Dev Tool">
  <meta name="twitter:description" content="Your product description">
  <meta name="twitter:image" content="https://yourdomain.com/images/twitter-card.jpg">
</head>
```

## Troubleshooting

### CSS not loading
- Ensure `index.css` is in the same directory as `index.html`
- Check the path in `<link rel="stylesheet" href="index.css">`
- Clear browser cache

### Dark theme not applying
- Verify the `theme-dark` class is on the `<html>` element, not `<body>`
- Check that CSS variables are defined for both `:root` and `.theme-dark`

### Images not displaying
- Verify image paths are relative: `images/photo.jpg` not `/images/photo.jpg`
- Check image file names match case-sensitive references
- Ensure images are committed to the repository

### Mobile menu not working
- Verify `index.js` is loaded before `</body>` closes
- Check console for JavaScript errors
- Ensure menu toggle button has correct class name

### Fonts not loading
- Host fonts locally in the `fonts/` directory
- Use `@font-face` declarations in CSS
- Avoid relying on external CDNs for production

## Best Practices

1. **Keep it simple**: Remove unused sections to maintain fast load times
2. **Optimize images**: Use WebP format and lazy loading
3. **Test responsiveness**: Check on multiple devices and screen sizes
4. **Validate HTML**: Use W3C validator to catch errors
5. **Add structured data**: Include JSON-LD for better SEO
6. **Monitor performance**: Use Lighthouse to maintain high scores
7. **Version control**: Commit changes incrementally with clear messages

## Additional Resources

- [Official LaunchKit website](https://launchkit.evilmartians.io/)
- [Color theme generator](https://codepen.io/romanshamin/full/QwWgNLN)
- [GitHub repository](https://github.com/evilmartians/devtool-template)
