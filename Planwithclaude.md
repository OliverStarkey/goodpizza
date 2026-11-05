# Pizza Shape Scorer: Project Plan

**Goal:** Take a photo of a pizza, extract an equation describing its perimeter, compare it to a dataset of preferred pizza shapes, and return a percentage likeness score.

A working prototype of the core pipeline already exists in `pizza_shape.py` (GrabCut segmentation → radial Fourier equation → similarity score against a perfect circle or a folder of reference photos). This plan covers how to turn it into something whose score is actually meaningful.

---

## Pipeline Overview

```
photo → segment pizza → outline → r(θ) Fourier equation → signature → compare to preferred shapes → % likeness
```

| Stage | Current approach | Possible upgrade |
|---|---|---|
| Segmentation | GrabCut (centered pizza assumed) | Segment Anything (SAM) or a fine-tuned U-Net |
| Perimeter equation | Polar Fourier series r(θ) | Elliptic Fourier descriptors (handles non-star-shaped outlines) |
| Normalization | Divide by mean radius, use harmonic amplitudes | Add perspective correction |
| Reference | Perfect circle, or mean of reference photos | Rating-weighted, per-style references |
| Score | `100·exp(-d/τ)` | Learned mapping from distance to human ratings |

---

## Phases

### Phase 1: Baseline (done / in progress)
- [x] Prototype script: segment, fit equation, score
- [ ] Set up repo, `requirements.txt`, and a README with usage
- [ ] Collect ~20 test pizza photos (top-down, varied shapes) to sanity check
- [ ] Verify the debug outline overlay looks right on every test image

### Phase 2: Robust Input Handling
- [ ] Swap GrabCut for SAM (or add it as an option) so the pizza doesn't need to be centered
- [ ] Detect and correct perspective (use the plate/box/table edges, or ask for top-down photos)
- [ ] Reject bad inputs (no pizza found, heavily tilted, multiple pizzas) with clear messages
- [ ] Unit tests with synthetic shapes (perfect circle, ellipse, square) where the right answer is known

### Phase 3: Define "Preferred Shape" (the hard part)
No standard dataset exists, so build one:
- [ ] Decide what's being measured: overall outline only, or outline per pizza style (round, rustic, rectangular, Detroit, etc.)
- [ ] Design a survey: show pairs or sets of pizza outlines/photos and have people rate or pick
- [ ] Collect responses (target a few hundred ratings at minimum; note demographics/region)
- [ ] Store ratings with the extracted signatures in a simple CSV or SQLite file
- [ ] Record the survey method and limitations in the repo

### Phase 4: Scoring That Means Something
- [ ] Replace the guessed `TAU` by fitting distance-to-rating on the survey data
- [ ] Try alternatives: rating-weighted average reference, nearest-neighbor to top-rated shapes, or a small regression model on the Fourier signature
- [ ] Hold out a test set and report how well the score predicts human ratings
- [ ] Pick the simplest method that performs well

### Phase 5: Interface and Release
- [ ] Simple web UI (Streamlit or Gradio): upload a photo, see the outline, equation, and score
- [ ] Optional: mobile-friendly capture flow
- [ ] Document results, limitations, and example outputs in the README
- [ ] Tag a v1.0 release

---

## Open Questions
1. Should "preferred" be one global ideal, or relative to pizza style?
2. How should irregular or partially eaten pizzas be treated?
3. Is the target the outline only, or should toppings and crust thickness ever matter?
4. How will survey participants be recruited, and how do I avoid a skewed sample?

## Risks
- **Score validity:** without real preference data the percentage only measures how close the shape is to the reference (e.g. circularity), not how liked it is.
- **Camera angle:** perspective distortion can make a round pizza look elliptical.
- **Segmentation failures:** cluttered backgrounds, dark plates, or cheese blending into the surface can break the outline.

## Suggested Repo Structure
```
pizza-shape-scorer/
├── README.md
├── PLAN.md
├── requirements.txt
├── pizza_shape.py
├── data/
│   ├── test_photos/
│   ├── preferred_pizzas/
│   └── survey_responses.csv
├── tests/
│   └── test_shapes.py
└── app.py            # UI (Phase 5)
```

## Success Criteria
- Correctly segments the pizza on at least 90% of test photos
- Score is invariant to pizza size and photo rotation
- Score correlates meaningfully with held-out human preference ratings
- Anyone can run it from the README in under five minutes
