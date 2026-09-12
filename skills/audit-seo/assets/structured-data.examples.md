# Structured Data Examples

Copyable, valid JSON-LD for the types dimension 4 (`references/rubric.md`) checks for most
often. Placeholder domains and values only — replace every field before use. Point a report
finding at the relevant block instead of writing markup from scratch in the Work Plan.

## Organization (site-wide, usually on the homepage)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Your Company",
  "url": "https://your.domain.com",
  "logo": "https://your.domain.com/logo.png",
  "sameAs": [
    "https://www.linkedin.com/company/your-company",
    "https://twitter.com/yourcompany"
  ]
}
</script>
```

## WebSite + SearchAction (site-wide, enables a sitelinks search box)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "url": "https://your.domain.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": "https://your.domain.com/search?q={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
</script>
```

## BreadcrumbList (any page more than one level deep)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://your.domain.com/" },
    { "@type": "ListItem", "position": 2, "name": "Blog", "item": "https://your.domain.com/blog" },
    { "@type": "ListItem", "position": 3, "name": "Post Title", "item": "https://your.domain.com/blog/post-title" }
  ]
}
</script>
```

## Article (blog posts, news)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "The Article's Actual Title",
  "image": ["https://your.domain.com/images/article-hero.jpg"],
  "datePublished": "2026-09-01T08:00:00-04:00",
  "dateModified": "2026-09-05T10:00:00-04:00",
  "author": { "@type": "Person", "name": "Author Name" },
  "publisher": {
    "@type": "Organization",
    "name": "Your Company",
    "logo": { "@type": "ImageObject", "url": "https://your.domain.com/logo.png" }
  }
}
</script>
```

`headline`, `image`, and `datePublished` are the properties Google treats as required — a
block missing any of them is a 🟡 at minimum, per rubric check 4.3.

## Product (only if the page actually shows a price and availability)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "image": "https://your.domain.com/images/product.jpg",
  "description": "A real, page-matching description of the product.",
  "offers": {
    "@type": "Offer",
    "url": "https://your.domain.com/products/example",
    "priceCurrency": "USD",
    "price": "49.00",
    "availability": "https://schema.org/InStock"
  }
}
</script>
```

**Never suggest this block for a page that doesn't actually display that price or
availability** — a mismatch between markup and visible content is check 4.4, a 🔴, and a
violation of Google's structured-data spam policies, not just a quality gap.
