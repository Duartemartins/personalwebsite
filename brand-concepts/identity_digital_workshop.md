---
layout: page
title: Brand Identity
permalink: /brand-identity/
nav_exclude: true
---

# Brand Identity: The Editorial Workshop

## Core Concept
"The Editorial Workshop" is a sophisticated evolution of the "Digital Bindery". It merges the authority and readability of high-end print journalism (The Free Press, Works in Progress) with the raw, functional aesthetic of a craftsman's workshop. It is intellectual, precise, and unapologetically digital-native while respecting print traditions.

## Visual Language
- **Metaphor**: The "Broadsheet" meets the "Workbench".
- **Key Elements**:
    - **Structural Borders**: Content is framed by distinct borders, separating the "reading area" from the "tools area".
    - **The "On The Desk" Sidebar**: A persistent left-hand column that acts as a workspace, holding navigation, branding, and tools, distinct from the content flow.
    - **Data-Led Storytelling**: Charts and data are first-class citizens, not afterthoughts.
- **Atmosphere**: Authoritative, Warm, Intellectual, Urgent.

## Typography
- **Display (Headlines)**: `Instrument Serif` (Serif).
    - Usage: Large, high-contrast headlines. It evokes the "Newspaper" feel but with a sharper, more modern cut.
- **Body (Long-form)**: `Libre Baskerville` (Serif).
    - Usage: Optimized for long-form reading. High x-height, open counters, traditional but screen-ready.
- **UI & Data (The "System")**: `Inter` (Sans-serif).
    - Usage: Navigation, metadata, chart labels, captions. It provides the "modern/technical" contrast to the serif content.

## Color Palette
- **Canvas (Paper)**: `#fcfbf9` (Warm Newsprint). A soft, off-white that reduces eye strain and mimics high-quality paper.
- **Sidebar (Desk)**: `#f4f3f0` (Slightly darker warm grey). Separates the "chrome" from the "content".
- **Ink (Primary Text)**: `#1a1a1a` (Near Black). Softer than pure black, mimicking dried ink.
- **Ink Light (Secondary)**: `#555555` (Dark Grey). For metadata and supporting text.
- **Accent (Editorial Red)**: `#c02626`. A deep, serious red used for emphasis, active states, and key data points.
- **Border**: `#e0ded9`. Subtle structural lines.

## Data Visualization System
Charts are built using pure CSS/HTML (or SVG for complex lines) to ensure lightweight performance and perfect scaling.

### 1. Bar Charts ("The Velocity Gap")
- **Structure**: Vertical bars within a defined grid.
- **Styling**:
    - Bars are solid `var(--ink)`.
    - Highlighted data points use `var(--accent)`.
    - Grid lines are subtle `var(--border)`.
- **Labels**:
    - X-Axis labels are rotated **45 degrees** (down-right) to accommodate long text without overlapping.
    - Labels are anchored to the bar's center but flow outwards.
- **Branding**:
    - Every chart must include a footer with:
        - **Left**: Source data citation.
        - **Right**: "Duarte Martins" and "@duarteosrm".

### 2. Line Charts ("Capital Divergence")
- **Structure**: SVG-based for smooth curves.
- **Styling**:
    - Lines are `stroke-width: 3`.
    - Primary trend: `var(--ink)`.
    - Divergent/Highlight trend: `var(--accent)`.
- **Labeling**:
    - **Direct Labeling**: Lines are labeled at their endpoints (no separate legends).
    - Y-Axis labels are aligned right, inside the grid area.

## Interactive Elements
- **Share Functionality**:
    - Located in the Sidebar or Footer.
    - **Twitter/X**: Opens a pre-filled tweet with Title + URL + "@duarteosrm".
    - **Copy Link**: Copies URL to clipboard and provides "Copied!" feedback state.
- **Reading Progress**: A subtle progress bar or "Read Time" indicator in the top metadata bar.

## GitHub Pages Compatibility
- **Theme Architecture**: The "Editorial Workshop" identity is implemented as a set of layout overrides (`_layouts/`) and a custom SCSS framework (`assets/css/style.scss`) on top of the standard Jekyll structure.
- **No Custom Plugins**: The design relies on standard Liquid templating and CSS, requiring no unsupported plugins, ensuring seamless deployment on GitHub Pages.
- **Asset Management**: All styles and scripts are self-contained within the repository, ensuring the visual identity remains consistent across local development and production environments.

## Responsive Behaviour
- **Philosophy**: The "Workshop" adapts to the screen size, prioritizing readability on smaller devices while maintaining the "Editorial" aesthetic.
- **Breakpoints**:
    - **Desktop (> 1024px)**: Full "Desk" layout with persistent sidebar and centered content.
    - **Tablet (768px - 1024px)**: Sidebar collapses into a top "Mobile Header". Content takes full width with generous padding.
    - **Mobile (< 768px)**:
        - **Typography**: Headlines scale down (`2.5rem`), body text remains legible.
        - **Navigation**: Accessible via a "Hamburger" menu in the sticky top header.
        - **Layout**: Single-column flow. Padding reduced to maximize reading area.
- **Mobile Navigation**:
    - A sticky top bar containing the Brand Name and a Menu Toggle.
    - Full-screen overlay menu for navigation links, ensuring touch targets are large and accessible.
