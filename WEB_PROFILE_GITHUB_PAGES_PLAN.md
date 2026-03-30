# Web Profile Design Plan for GitHub Pages

## 1) Project Goal
- Build a clean, modern personal profile website that is easy to maintain.
- Clearly present identity, skills, projects, and contact details.
- Ensure strong mobile and desktop experience.
- Host on GitHub Pages with a simple static setup.

## 2) Product Direction
- Site type: one-page scrolling profile website.
- Target audience: recruiters, collaborators, and tech community peers.
- Visual tone: confident, minimal, readable, and personal.

## 3) Page Structure
1. Hero
- Avatar/photo, full name, and role title.
- One short personal tagline (10-15 words).
- Primary CTA buttons: View Projects and Contact.

2. About
- Short intro paragraph (3-5 sentences).
- Personal values and work style.

3. Skills
- Group by category: Frontend, Backend, DevOps, Tools.
- Optional proficiency labels (Beginner / Intermediate / Advanced).

4. Projects
- Show 3-6 highlighted projects.
- Each card includes: name, one-line summary, stack, demo link, source link.

5. Experience or Learning Roadmap
- If experienced: timeline of roles and outcomes.
- If early career: learning roadmap + completed milestones.

6. Contact
- Email, GitHub, LinkedIn, optional YouTube/Facebook.
- Optional CV download button (PDF).

7. Footer
- Copyright and current year.

## 4) UI and UX Direction
- Typography: max 2 fonts, readable hierarchy.
- Color system: one primary color, one accent color, neutral base.
- Layout:
- Desktop: centered content, max width around 1100-1200px.
- Mobile: single-column stacking with clear spacing.
- Motion:
- Light page entrance transitions (fade or slide).
- Hover feedback for buttons and project cards.
- Navigation:
- Sticky navbar with anchor links (for example #about and #projects).
- Back-to-top button for long mobile scrolling.

## 5) Technical Stack for GitHub Pages
- Plain HTML, CSS, and JavaScript.
- Recommended folder layout:
- index.html
- assets/css/style.css
- assets/js/main.js
- assets/img/
- assets/docs/CV.pdf
- No backend required.
- For contact forms, use Formspree or Getform, or a mailto fallback.

## 6) Quality Standards
- Performance:
- Use WebP or AVIF image formats.
- Compress images and lazy-load below-the-fold media.
- SEO basics:
- Meta title and description.
- Open Graph tags for social sharing.
- Favicon.
- Accessibility:
- Good color contrast.
- Alt text on all meaningful images.
- Keyboard-friendly navigation.
- Responsive testing breakpoints:
- 360px, 768px, 1024px, and 1280px+.

## 7) Suggested Execution Plan (5 Sessions)
1. Session 1 - Content lock
- Finalize name, tagline, short bio.
- Prepare avatar, social links, and project list.

2. Session 2 - Layout foundation
- Build Hero, About, Skills, Projects, and Contact sections.
- Implement baseline responsive behavior.

3. Session 3 - Visual polish
- Improve spacing, color balance, typography, and card design.
- Add icons and subtle interactions.

4. Session 4 - Optimization pass
- Improve performance and accessibility.
- Add SEO metadata, OG tags, and favicon.

5. Session 5 - Deployment
- Push to main branch.
- Enable GitHub Pages and validate production URL.

## 8) GitHub Pages Deployment Steps
1. Create or use an existing GitHub repository.
2. Naming options:
- username.github.io for the main personal site.
- any-repo-name for a project site.
3. Push all website files to the main branch.
4. In GitHub: Settings > Pages.
- Source: Deploy from a branch.
- Branch: main, folder: /(root).
5. Save and wait 1-3 minutes.
6. Open the public Pages URL and verify all links and assets.

## 9) Definition of Done
- [ ] Content is clear, concise, and typo-free.
- [ ] Layout works well on both desktop and mobile.
- [ ] Social and contact links work.
- [ ] Project cards include summary, stack, and links.
- [ ] SEO tags and sharing preview are configured.
- [ ] Site is successfully deployed on GitHub Pages.

## 10) Optional Next Improvements
- Add a mini blog page (for example posts.html).
- Add dark/light theme toggle.
- Add analytics (Google Analytics or Plausible).
- Add a custom domain with CNAME.

## 11) Handoff Prompt (for another dev or AI)
Copy this block and give it to the person who will build the site:

```text
Build a one-page personal profile website using plain HTML, CSS, and JavaScript.

Requirements:
1) Sections: Hero, About, Skills, Projects, Experience or Learning Roadmap, Contact, Footer.
2) Responsive design for 360px, 768px, 1024px, and 1280px+.
3) Modern and clean visual style, with subtle entrance and hover animations.
4) Sticky navbar with anchor links and smooth scrolling.
5) Project cards must include title, one-line summary, tech stack, demo link, and source link.
6) SEO basics: title, description, Open Graph tags, favicon.
7) Accessibility: alt text, keyboard-friendly navigation, readable contrast.
8) File structure:
	- index.html
	- assets/css/style.css
	- assets/js/main.js
	- assets/img/
	- assets/docs/CV.pdf (optional)
9) Keep code clean and easy to edit.
10) Prepare for GitHub Pages deployment from main branch root.

Deliverables:
- Complete static site files.
- Short README with deploy steps.
```