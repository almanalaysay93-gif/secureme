# **UX Laws and Human-Computer Interaction (HCI) Frameworks**

Human-Computer Interaction (HCI) and digital product design rely on established psychological principles, behavioral models, and interaction laws. Designing digital products without accounting for these fundamental laws increases cognitive overhead, slows task completion, and increases abandonment rates. This document details foundational behavioral and cognitive interaction laws, providing operational definitions, practical UX applications, and concrete real-world examples for each.

## **1\. Foundational Interaction & Usability Laws**

| Principle / Law | Core Rule & Definition | Real-World Examples & UX Applications   |
| :---- | :---- | :---- |
| **Jakob’s Law** | Users spend most of their time on other websites, meaning they expect your interface to function similarly to the conventions they already know. | **E-commerce Shopping Carts:** Placing the cart icon in the top-right corner with a numeric badge indicator. **Search Bars:** Positioning the search bar prominently at the top-center or top-left of navigation headers. **Form Navigation:** Placing "Cancel" on the left and "Submit/Save" on the right in modal dialogue footers. |
| **Hick’s Law (Hick-Hyman Law)** | The time required to make a decision increases logarithmically with the number and complexity of choices available: *T \= b · log2(n \+ 1\)*. | **Multi-Step Checkout Wizards:** Splitting checkout into Shipping, Payment, and Review rather than presenting a 30-field single form. **Categorized Mega-Menus:** Grouping 100+ site links into 4–6 high-level categories with sub-menus. **Smart Defaults / Recommended Tiers:** Pre-selecting a "Recommended" pricing plan to reduce decision paralysis among 5 options. |
| **Fitts’s Law** | The time required to rapidly move to a target area is a function of the ratio between the distance to the target and the width of the target. | **Mobile Bottom Navigation:** Placing primary app tabs along the bottom screen edge within natural thumb reach. **Sticky CTA Bars:** Keeping "Add to Cart" or "Book Now" anchored to the screen bottom during mobile scrolling. **Minimum Touch Targets:** Sizing interactive buttons to at least 48x48px with 8px+ padding to prevent accidental mis-taps. |
| **Miller’s Law** | The average person can only keep 7 (± 2\) items in their working memory simultaneously. | **Credit Card Formatting:** Formatting 16-digit card inputs into 4 discrete chunks of 4 digits (4444 4444 4444 4444). **Settings Organization:** Grouping dozens of application settings into 5–7 distinct panels (e.g., Account, Privacy, Billing). **Phone Number Formatting:** Breaking 10-digit numbers into area codes and prefixes (e.g., (555) 019-2834). |
| **Tesler’s Law (Conservation of Complexity)** | Every system contains an irreducible amount of complexity. The only choice is whether the system absorbs it or forces the user to handle it. | **Address Autofill:** Populating city, state, and country automatically once the user enters a zip/postal code. **One-Click Social Login:** Handling OAuth token exchange automatically instead of requiring manual account verification. **Card Type Auto-Detection:** Determining Visa, Mastercard, or Amex directly from the starting digits rather than asking via dropdown. |
| **Doherty Threshold** | Productivity increases significantly when computer and user interact at a pace that ensures neither has to wait more than 400 milliseconds. | **Skeleton Loading Screens:** Displaying wireframe shimmer placeholders instantly while background API data loads. **Optimistic UI Updates:** Toggling a "Like" button heart color immediately before the network confirmation completes. **Instant Search Typeahead:** Displaying matching query suggestions within 150ms of a keystroke. |
| **Von Restorff Effect (Isolation Effect)** | When multiple similar items are present, the item that differs visually from the rest is most likely to be remembered. | **"Most Popular" Pricing Card:** Elevating the middle subscription card with a badge, larger border, and bold accent color. **Destructive Action Modals:** Styling the "Delete Project" button in bright red against muted gray secondary buttons. **Notification Badges:** Displaying high-contrast red dot indicators on top of neutral monochromatic navigation icons. |
| **Zeigarnik Effect** | People remember uncompleted or interrupted tasks significantly better than completed ones. | **Profile Completion Gauges:** Showing "Your profile is 70% complete" with a progress circle to drive additional data input. **Gamified Onboarding Checklists:** Displaying 3 out of 5 checked items to incentivize first-time workspace setup. **Multi-Lesson Course Trackers:** Showing a partially filled progress ring next to in-progress learning modules. |

## **2\. Cognitive & Perceptual Frameworks**

### **A. Mental Models & Skeuomorphic Cues**

A mental model is what the user believes about the system at hand based on previous experiences. When digital systems map directly to familiar physical counterparts, user onboarding friction drops significantly.

> * **Toggle Switch UI:** Mimicking physical wall light switches for boolean on/off configurations.  
> * **Trash Can / Recycle Bin:** Using a wastebasket icon for staged item deletion before permanent removal.  
> * **Digital Document Folders:** Organizing file systems into visual nested folders to represent hierarchical disk directories.

### **B. Gestalt Principles of Visual Grouping**

Gestalt psychology explains how human visual perception organizes elements into unified structures rather than disparate parts. Key examples across core Gestalt principles include:

> * **Law of Proximity (Form Labeling):** Placing input labels 4px directly above their corresponding text field while maintaining 24px spacing before the next field to avoid misattribution.  
> * **Law of Common Region (Card Containers):** Enclosing related product images, titles, pricing, and buy buttons inside a bordered white card against a light gray background.  
> * **Law of Similarity (Interactive States):** Styling all hyperlink text in blue underline and all primary actions in solid navy to clearly distinguish clickable roles.

### **C. Cognitive Load Theory in Interface Architecture**

Human working memory has limited processing bandwidth. Cognitive load is managed across three distinct dimensions:

> * **Managing Intrinsic Load (Task Clarity):** Providing plain-language tooltips and inline explanations for complex financial or medical calculations.  
> * **Eliminating Extraneous Load (Noise Reduction):** Removing unneeded decorative borders, flashing banners, and duplicate navigational links from core dashboard views.  
> * **Promoting Germane Load (Pattern Recognition):** Using consistent layout templates and reusable design system components across all sub-pages so users quickly build lasting mental models.