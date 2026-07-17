# Recognizing Faces in the Wild — Setup & Data Guide

Welcome! This exercise uses a dataset of family photos. Your task: build a model
that looks at two face photos and predicts whether the two people are blood
relatives.

Read this whole doc before you start writing code — most early confusion on this
dataset comes from mixing up "same folder" with "related."

---

## 1. What you've been given

You should have received:

```
train/                     (training images)
train_relationships.csv    (training labels)
test/                       (test images)
sample_submission.csv      (submission format)
```

That's everything you need. Put it all in one working folder.

---

## 2. The most important thing to understand first

> **Being in the same family folder does NOT mean two people are related.**

A husband and wife live in the same family folder, but share no blood relationship.
Only pairs explicitly listed in `train_relationships.csv` count as "related." Keep
this in mind or your labels will be wrong from step one.

---

## 3. How the training data is organized

```
train/
  F0002/
    MID1/
      photo1.jpg
      photo2.jpg
    MID2/
      photo1.jpg
    MID3/
      photo1.jpg
      photo2.jpg
  F0003/
    ...
```

- **`F####`** = a family ID. Everyone under one `F####` folder belongs to the same
  extended family.
- **`MID#`** = a specific person within that family (Member ID 1, 2, 3...). All
  photos inside one `MIDx` folder are the same person.

So the structure reads as: **family → person → that person's photos.**

### `train_relationships.csv`
This file lists *related people*, not related photos:

| p1 | p2 |
|---|---|
| F0002/MID1 | F0002/MID3 |
| F0002/MID2 | F0002/MID3 |

Each row = a **positive** example (these two people are kin). Important
consequences:

- Only positive pairs are given — there are **no negative rows**.
- You must build your own negative pairs (e.g. by pairing people from different
  families, which guarantees "unrelated").
- The dataset is naturally imbalanced toward "unrelated" once you consider all
  possible pairs — this is why accuracy alone is a weak metric here, and you should
  think in terms of ranking quality (AUC-ROC) instead.
- A training example is an **image pair**, not a person pair — once you know two
  people are related, pick one photo from each of their folders to form the actual
  input to your model.

---

## 4. How the test data is organized

```
test/
  abcdef.jpg
  ghijkl.jpg
  ...
```

Test images are flat — no family/person folders, no names.

**You only need to run inference on the exact pairs listed in
`sample_submission.csv`.** You do not need to (and should not try to) generate or
guess additional pairs from `test/` on your own — the pairs you're graded on are
fixed and given to you:

```
img_pair,is_related
abcdef-ghijkl,0
mnopqr-stuvwx,1
```

- `img_pair`: two filenames joined by a hyphen (`abcdef-ghijkl` = `abcdef.jpg` +
  `ghijkl.jpg`).
- `is_related`: your prediction. Submit a probability between 0 and 1 rather than
  a hard 0/1 — this gives partial credit for good ranking rather than an all-or-
  nothing cutoff.

---

## 5. Suggested workflow

1. Load `train_relationships.csv` → these are your positive person-pairs.
2. Generate negative person-pairs yourself (people from different families).
3. For each person-pair, sample photo(s) from their respective `MID#` folders to
   build actual image-pair training examples.
4. Train a model that outputs a "related" probability for an image pair (a
   face-embedding / siamese-network approach is the standard starting point).
5. For every pair listed in `sample_submission.csv`, load the two corresponding
   images from `test/` and run your model on them.
6. Fill in your predicted probability for each row and submit the CSV in the same
   format.

---

## 6. Quick reference

| File / folder | What it is |
|---|---|
| `train/` | Training photos, organized `family/person/photo.jpg` |
| `train_relationships.csv` | List of related person-pairs (positives only) |
| `test/` | Unlabeled test photos |
| `sample_submission.csv` | The exact pairs to predict, and the format to submit in |

If anything in the data still looks confusing once you start poking around the
actual files, that's a good sign to ask rather than guess — small mistakes here
(like assuming same-folder means related) will quietly wreck your labels.
