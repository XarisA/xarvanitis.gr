# Rebuilding My Personal Site in Angular - From Hugo Coder to a Modern, Dynamic Platform

When I set out to rebuild my developer hub, meaning this site, I wanted more than just a place to showcase my work. I envisioned a dynamic platform that would reflect who I am as a developer while also challenging me to expand my skillset.

For years, I used Hugo with Coder theme. It was clean, minimal, and served its purpose, but over time, the limitations became clear. My old site [xaris.mooo.com](http://xaris.mooo.com) worked, but it was static, fragmented, and increasingly hard to maintain as my projects, blog posts, and resume grew. I wanted something cleaner, more unified, and easier to extend.

What started as a simple "let’s refresh my resume page" project quickly evolved into a full-fledged portfolio platform: blog, project pages, Markdown-powered content, reusable Angular components, smooth navigation, and a crisp layout inspired by Hugo Coder but fully rebuilt with Angular.

---

## The Why, The What, and The How

A blog is essential for anyone with something to say and a desire to express themselves, but why build one from scratch when there are existing platforms and frameworks?

Well… because I’m a developer. I already had a personal site built with Hugo, but now I wanted to push my boundaries. In 2024, I transitioned into a full-stack developer role, and I realized that the best way to truly master Angular and expand my understanding, was to get hands-on experience and write "miles of code".

Building from scratch forces you to wrestle with questions like: *How should this be done? Is this the recommended approach? Why does one method work better than another?* These challenges push you to dig deeper, explore alternatives, and ultimately grow as a developer.

I’ve found that while tutorials and videos are convenient, they’re often less effective and more time-consuming. I prefer reading documentation and guides, which are frequently updated and reflect the latest changes. Books remain valuable for foundational or theoretical concepts, but for frameworks and tools, up-to-date docs is the way to go.

This approach isn’t easy, it’s challenging, but it’s crucial for avoiding "hitting the wall" as the runners say. Comfort comes from sticking to familiar tools, but technology moves fast, and staying curious and adaptable is essential.

>The illiterate of the 21st century will not be those who cannot read and write, but those who cannot learn, unlearn, and relearn.


## Taking Hugo Coder Theme as Inspiration and Improving It

Hugo Coder taught me the value of clean typography, minimalism, and subtle color schemes. I loved the look, but I wanted a site I could *own and mold freely*. So instead of tweaking the old theme, I rebuilt it from scratch using Angular, Material UI, and Markdown rendering.

The new site keeps the spirit of Hugo Coder centered layouts, tidy typography, and minimalism, but adds the flexibility of a modern framework. Every part of the site, from the toolbar to scroll arrows, is now an Angular component. That means smoother transitions, reusable modules, and far more room to experiment.

Some positive changes compared to [the old site](http://xaris.mooo.com): faster load times thanks to lazy loading, Markdown-driven projects and blog posts that are effortless to update, better mobile responsiveness, SEO-friendly resume rendering, and thoughtful UX touches like a floating back button, scroll-to-top arrow, and print-friendly resume.


## A Modular Architecture That Just Works

The backbone of the new site is modular. I split it into feature modules Home, Resume, Projects, Blog, Shared, so each part manages itself. Lazy loading ensures Angular only fetches the code when needed, keeping the main bundle small and the app snappy.

Adding a new blog post or project is now as simple as creating a Markdown file and updating an index. No Angular code required. Combined with hosting on GitHub Pages, this setup is fully automated: I just push new Markdown files to the repository, and everything works out-of-the-box.


## My Resume, Just another JSON file

The resume is entirely JSON-driven. All sections Experience, Education, Skills, ...  are structured in one place. Angular reads the file at runtime, so updating my resume doesn’t require touching code.

Even complex layouts, like multiple roles within a company or neatly aligned dates, are handled dynamically. Compared to the old Hugo Coder site, this makes content management vastly simpler while remaining SEO-friendly.


## Markdown for Projects and Blog Posts

Projects and blog posts now live in Markdown files, just like content in a static generator, but rendered dynamically in Angular. Each project has its own file, with metadata driving the project index. Clicking a project dynamically loads its content.

For the blog, the workflow is the same: metadata drives summaries, dates, and titles, and the Markdown content renders beautifully when opened. Writing, updating, or publishing content is effortless with no Angular code changes needed. Push to GitHub, and it’s live.


## Clean, Flexible Icon System

Navigation icons, print buttons, and external link indicators come from Angular Material. Brand logos like GitHub and LinkedIn use Font Awesome. I only import the icons I need, keeping the bundle light while maintaining full control over the visuals.


## Fast, Snappy Navigation

Lazy loading makes a noticeable difference. Each section Resume, Projects and Blog, loads independently, and Markdown files only load on demand. Navigation feels smooth and app-like, a clear upgrade over the old static Hugo Coder site.


## Thoughtful UX Enhancements

Small touches make a big difference: a floating back button, a scroll-to-top arrow, and a print-friendly resume view. These enhancements address minor annoyances from the old site, creating a smoother, more professional experience.

---

## Looking Back and Forward

Rebuilding my site in Angular has been both a fun challenge and a huge upgrade over years of Hugo Coder. The new portfolio is not just a personal website, it’s a fully functional, dynamic content platform: modular, scalable, fast, and simple to maintain.

Hugo Coder’s aesthetic clarity remains, but now it comes with Angular’s power: JSON-driven resume, Markdown content rendering, lazy-loaded modules, and a cleaner, more modern interface all fully automated on GitHub Pages.

This rebuild transformed something static into something alive.. a site I can update, grow, and refine without friction, while keeping the minimalist style I love.

Something I could explore in the future is using Firebase to store my Markdown files instead of keeping them in the GitHub repository. This would allow me to update content dynamically without pushing changes to GitHub, while still hosting the site statically on GitHub Pages. For now, this isn’t necessary because the current setup works smoothly and keeps the workflow simple, but it could be an interesting next step to explore for a more dynamic content pipeline.
