# Modular Design

{{< callout context="note" >}}These docs are written for developers. New to the badge? Check out our [blog](/blog/) for guides and project walkthroughs to get started.{{< /callout >}}

The badge is built as a **three-layer stack** that screws together, so each layer can be swapped or replaced without touching the others. The layers from front to back are: faceplate, main PCB, and backplate.

```
┌─────────────────────┐
│     Faceplate       │  ← custom art, expansion PCB
├─────────────────────┤
│     Main PCB        │  ← all onboard components
├─────────────────────┤
│     Backplate       │  ← lanyard clip / badge mount
└─────────────────────┘
```

**The Design Files →** [github.com/LyndLabs/Edge-AI-Badge]()

## Stackup + Expansion
`more coming soon`