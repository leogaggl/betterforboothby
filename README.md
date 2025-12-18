# Better for Boothby Website

This repository contains the source code for the "Better for Boothby" campaign website, a community-driven movement for better political representation in the Australian Federal Electorate of Boothby.

Published site: [https://betterforboothby.com.au/](https://betterforboothby.com.au/)

The website is built using the [Hugo Static Site Generator](https://gohugo.io/) and the [Hugo Universal Theme](https://github.com/devcows/hugo-universal-theme).

## Development

To run the website locally for development, you will need to have Hugo installed.

1.  **Clone the repository and initialise submodules:**

    There are two ways to clone the repository and ensure submodules are correctly initialised:

    *   **Option 1: Clone with `--recurse-submodules` (recommended):**
        ```bash
        git clone --recurse-submodules https://github.com/<your-username>/betterforboothby.git
        cd betterforboothby
        ```

    *   **Option 2: Clone normally, then initialise submodules:**
        ```bash
        git clone https://github.com/<your-username>/betterforboothby.git
        cd betterforboothby
        git submodule update --init --recursive
        ```

2.  **Run the Hugo server:**
    ```bash
    hugo server
    ```

3.  **Open the site** in your browser at `http://localhost:1313`.

## Deployment

The site is automatically deployed to GitHub Pages via a GitHub Actions workflow whenever changes are pushed to the `main` branch.

## Image Guidelines

To ensure optimal display quality and performance, please adhere to the following image size guidelines:

### Front-page Feature Images

These images are displayed in a four-column grid on the homepage and are vertically constrained.

*   **Recommended Size:** 450 pixels wide by 300 pixels high (3:2 aspect ratio).
*   **Notes:** This size is optimized for high-resolution displays and will be scaled down to a maximum display height of 150px. Using a consistent size and aspect ratio for all feature images will ensure a clean and consistent layout.

### Blog Post Header Images

These images serve as banners for individual blog posts.

*   **Recommended Size:** 2400 pixels wide by 1200 pixels high (2:1 aspect ratio).
*   **Notes:** This larger size is ideal for high-resolution screens. A minimum size of 1200 pixels wide by 600 pixels high (2:1 aspect ratio) is acceptable for standard displays.

