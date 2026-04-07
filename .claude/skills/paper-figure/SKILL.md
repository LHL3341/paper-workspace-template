---
name: paper-figure
description: Generate publication-ready scientific figures for ML papers using matplotlib. Covers architecture diagrams, result tables, bar/line charts, ablation plots, and multi-panel layouts with colorblind-safe palettes.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash
argument-hint: "[figure_type] [data_source]"
arguments: figure_type,data_source
---

# Paper Figure — Publication-Quality Scientific Visualization

## Overview

Generate publication-ready figures for ML/AI papers. Follows best practices from scientific visualization guidelines and top-venue formatting standards (NeurIPS, ICML, ICLR).

## When to Use

- Creating result comparison charts (bar, line, grouped)
- Drawing architecture/pipeline diagrams
- Building ablation study tables or heatmaps
- Generating multi-panel figure layouts
- Making figures for paper submission

## Figure Design Principles

1. **Every figure must have a clear, self-contained message.** A reader should understand the figure without reading the full paper.
2. **Minimize chartjunk.** Remove gridlines, background shading, 3D effects, and unnecessary borders.
3. **Use direct labeling** instead of legends when possible.
4. **Remove top and right spines** for cleaner appearance.
5. **All text must be readable at print size** — minimum 7pt font at final figure width.
6. **Figure 1 is the most important.** It should convey the key idea or main result at a glance. Invest significant effort here.

## Figure Sizing (NeurIPS)

| Type | Width | Usage |
|------|-------|-------|
| Single column | 5.5 in (140 mm) | Most figures |
| Full width | 7.1 in (180 mm) | Architecture diagrams, multi-panel |

- Resolution: 300 DPI minimum for raster; prefer vector (PDF, SVG)
- Always use `plt.savefig(..., bbox_inches='tight', dpi=300)`

## Colorblind-Safe Design

1. **Required palettes**: Use `tab10`, seaborn `colorblind`, Okabe-Ito, or viridis/cividis
2. **Never rely on color alone** — combine with hatching, markers, or line styles
3. **Avoid red-green** combinations; prefer blue-orange or blue-yellow
4. Test with colorblind simulator before submission

## Standard Figure Templates

### Template 1: Result Comparison Bar Chart

```python
import matplotlib.pyplot as plt
import numpy as np

# NeurIPS style setup
plt.rcParams.update({
    'font.size': 9,
    'font.family': 'serif',
    'axes.linewidth': 0.8,
    'axes.spines.top': False,
    'axes.spines.right': False,
    'figure.dpi': 300,
})

fig, ax = plt.subplots(figsize=(5.5, 3.0))

methods = ['Baseline', 'SFT', '+ Dedup', '+ Distill']
acc = [75.73, 79.33, 77.91, 78.35]
colors = ['#4e79a7', '#59a14f', '#e15759', '#f28e2b']  # colorblind-safe

bars = ax.bar(methods, acc, color=colors, width=0.6, edgecolor='white', linewidth=0.5)
ax.set_ylabel('Accuracy (%)')
ax.set_ylim(74, 81)

# Direct labeling
for bar, v in zip(bars, acc):
    ax.text(bar.get_x() + bar.get_width()/2, v + 0.2, f'{v:.1f}',
            ha='center', va='bottom', fontsize=8)

plt.tight_layout()
plt.savefig('figures/result_comparison.pdf', bbox_inches='tight', dpi=300)
```

### Template 2: Multi-Panel Ablation

```python
fig, axes = plt.subplots(1, 3, figsize=(7.1, 2.5), constrained_layout=True)

for i, (ax, title) in enumerate(zip(axes, ['(A) Dedup Effect', '(B) Distill Effect', '(C) Combined'])):
    ax.set_title(title, fontsize=9, fontweight='bold', loc='left')
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)
    # ... plot data ...

plt.savefig('figures/ablation_panels.pdf', bbox_inches='tight', dpi=300)
```

### Template 3: Architecture/Pipeline Diagram

For architecture diagrams, prefer:
1. **TikZ** in LaTeX for maximum control and vector quality
2. **matplotlib patches + arrows** for quick prototyping
3. **draw.io → export SVG** for complex diagrams

```python
# Quick pipeline diagram with matplotlib
fig, ax = plt.subplots(figsize=(7.1, 1.8))
ax.axis('off')

boxes = ['Diagnose', 'Plan', 'Generate', 'Verify', 'Train', 'Evaluate']
colors = ['#4e79a7', '#4e79a7', '#59a14f', '#e15759', '#f28e2b', '#f28e2b']

for i, (name, c) in enumerate(zip(boxes, colors)):
    x = i * 1.15
    rect = plt.Rectangle((x, 0.3), 1.0, 0.4, facecolor=c, alpha=0.8,
                          edgecolor='white', linewidth=1.5, zorder=2)
    ax.add_patch(rect)
    ax.text(x + 0.5, 0.5, name, ha='center', va='center',
            fontsize=8, color='white', fontweight='bold', zorder=3)
    if i < len(boxes) - 1:
        ax.annotate('', xy=(x + 1.1, 0.5), xytext=(x + 1.0, 0.5),
                    arrowprops=dict(arrowstyle='->', color='gray', lw=1.5))

ax.set_xlim(-0.2, len(boxes) * 1.15)
ax.set_ylim(0, 1)
plt.savefig('figures/pipeline.pdf', bbox_inches='tight', dpi=300)
```

### Template 4: Subskill Gap Analysis (Horizontal Bar)

```python
fig, ax = plt.subplots(figsize=(5.5, 2.5))

skills = ['Material sel.', 'Physical prop.', 'Causation', 'Physical seq.']
train_pct = [19.8, 0.5, 3.2, 15.4]
error_pct = [31.2, 6.9, 4.6, 6.4]

y = np.arange(len(skills))
h = 0.35
ax.barh(y - h/2, train_pct, h, label='Train %', color='#4e79a7')
ax.barh(y + h/2, error_pct, h, label='Error %', color='#e15759')
ax.set_yticks(y)
ax.set_yticklabels(skills)
ax.set_xlabel('Proportion (%)')
ax.legend(frameon=False, fontsize=8)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

plt.tight_layout()
plt.savefig('figures/subskill_gap.pdf', bbox_inches='tight', dpi=300)
```

## Multi-Panel Layout Rules

1. Label panels with bold uppercase: **(A)**, **(B)**, **(C)** in top-left
2. Use consistent axis scales across panels when comparing
3. Share axes where appropriate (`sharey=True`)
4. Maintain consistent font sizes and line widths across all panels
5. Use `constrained_layout=True` for automatic spacing

## Statistical Annotations

1. Show individual data points alongside summaries (strip + box plots)
2. Always specify error bar type in caption: SEM, SD, or 95% CI
3. Significance brackets: \* p<.05, \*\* p<.01, \*\*\* p<.001
4. Never use bar charts for small-n data — use dot plots

## Export Checklist

- [ ] Vector format (PDF) for all line art
- [ ] Font embedded or converted to outlines
- [ ] Axis labels include units: "Time (s)", "Accuracy (%)"
- [ ] Caption fully explains all symbols and panels
- [ ] Colors match between figure and caption/legend
- [ ] Tested in grayscale
- [ ] Readable at final print width

## References

- Adapted from [AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw) scientific-visualization skill
- Adapted from [K-Dense-AI/claude-scientific-skills](https://github.com/K-Dense-AI/claude-scientific-skills)
- Tufte, E. "The Visual Display of Quantitative Information" (2001)
