---
title: "Circuit Board"
description: "PCB layout and design files for the Edge AI Foundation Badge."
summary: ""
date: 2026-06-09T00:00:00+00:00
lastmod: 2026-06-09T00:00:00+00:00
draft: false
weight: 111
toc: false
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

{{< callout context="note" >}}These docs are written for developers. New to the badge? Check out our [blog](/blog/) for guides and project walkthroughs to get started.{{< /callout >}}

The badge is a 4-layer PCB designed in KiCad.  The stackup prioritizes a clean ground plane under the RF section and keeps high-speed SPI traces short to the display.  The round form factor is driven by the logo — every layer is circular to match.

## Design Files 
Design Files → **[github.com/LyndLabs/Edge-AI-Badge](https://github.com/LyndLabs/Edge-AI-Badge)**