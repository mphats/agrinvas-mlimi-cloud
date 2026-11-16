# Model provenance and limitations

Generated: 2026-11-15T18:32:47.636247+00:00  
Code revision: `unknown`

This project uses a mix of in-house trained models and rule-based components. External APIs are listed separately and are not presented as in-house models.

## Image dataset composition

- Total images: 1546 across 30 classes
- Malawian-sourced: 211 (13.65%)
- By country: Malawi=211, unreported=1335
- By source: gbif_malawi=41, inaturalist=634, inaturalist_malawi=71, madiphs_ckan_pictorial=92, moa_malawi_faw_pictures=7, unreported=701
- Licence declarations: CC BY-NC (MaDiPHS / MoA Malawi)=7, Creative Commons Attribution Share-Alike=92, cc-by=94, cc-by-nc=511, cc-by-nc-nd=19, cc-by-nc-sa=25, cc-by-sa=6, cc0=21, http://creativecommons.org/licenses/by-nc/4.0/=38, http://creativecommons.org/licenses/by-sa/4.0/=1, http://creativecommons.org/licenses/by/4.0/=2, unknown=730

Malawian images come from CKAN MaDiPHS/Ministry of Agriculture pictorial guides, GBIF Malawi field observations, iNaturalist Malawi, and the MoA 'Pictures of FAW in maize' archive. Images without per-file provenance predate the provenance audit and are marked `unreported`.

## In-house models

### pest_cnn

- Version: `cnn-resnet18-20261115`
- Architecture: ResNet18 (ImageNet IMAGENET1K_V1) + custom 2-layer head
- Base weights: torchvision resnet18 IMAGENET1K_V1 (BSD-3-Clause)
- Dataset manifest: `data/pest_images_manifest.json`
- Evaluation report: `models/pest_cnn/evaluation_report.json`
- Licence: Model weights: project-owned. Base ResNet18 ImageNet weights: BSD-3-Clause via torchvision.
- Limitations: Small, imbalanced image set; 211 of 1546 images (13.65%) are explicitly Malawian-sourced; the rest are global. Held-out accuracy is reported in models/pest_cnn/evaluation_report.json. Use as a screening aid only.

Evaluation note: The test split changed when Malawian images were added (231 test images vs 144 previously), so the previous 0.5417 figure is not comparable. The previous production model was re-evaluated on the new split: accuracy 0.3983 and top-3 0.5584. The current model scores 0.4372 and 0.6494 on the same split.

Metrics: best_val_acc=0.3695652173913043, best_val_loss=2.235176995396614, test_acc=0.43722943722943725, num_train_images=1085, num_val_images=230, num_test_images=231

### pest_diagnosis_multilingual

- Version: `distilbert-multilingual-ckan-2026`
- Architecture: DistilBertForSequenceClassification
- Base weights: distilbert-base-multilingual-cased (Apache-2.0)
- Dataset manifest: `data/text_training_manifest.json`
- Evaluation report: `not available`
- Licence: Base model: Apache-2.0. Fine-tuned weights: project-owned, but training text is CKAN MaDiPHS/MoA content under per-dataset licences (often non-commercial).
- Limitations: Trained on 23 records; not loaded on demand in production because loading it OOM-killed the gunicorn worker.

### text_classifier

- Version: `v2`
- Architecture: TF-IDF word(1,2) + char_wb(3,5) -> LogisticRegression
- Base weights: n/a
- Dataset manifest: `data/text_training_manifest.json`
- Evaluation report: `models/text_classifier/current/evaluation_report.json`
- Licence: Project-authored templates plus CKAN MaDiPHS/MoA text under per-dataset licences (often non-commercial). See PROVENANCE.csv.
- Limitations: Most training rows are synthetic templates derived from the knowledge base; real-text holdout performance is much lower. Confidence is indicative.

Metrics: accuracy=0.9695290858725761, macro_f1=0.9700744494367683, weighted_f1=0.9701573456987164, top3_accuracy=0.9916897506925207, n_train=1850, n_test=361

### rule_based

- Version: `code-unknown`
- Architecture: Heuristic colour/texture image rules + bilingual rule-based pest chat
- Base weights: n/a
- Dataset manifest: `n/a`
- Evaluation report: `not available`
- Licence: Project code and knowledge base; project-owned.
- Limitations: Not a learned model. Image fallback confidence is capped at 0.75. The pest chat assistant is rule-based and does not call an LLM.

## External services (not in-house models)

- PlantVillage API (gated off unless USE_EXTERNAL_PLANTVILLAGE=true)
- OpenWeatherMap API
- Open-Meteo API
- NASA GIBS
- OpenAI API (key placeholder, unused)

## Reproduction

```bash
# Image model
venv/bin/python -m ai.cnn_trainer

# Text classifier
venv/bin/python ai/train_text_model.py

# Manifests
venv/bin/python ai/generate_manifests.py
```
