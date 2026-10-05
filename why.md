
### 17.1 Primary PAD Datasets

| Dataset | Access Link | Access Type | Verified? | What it validates | What it cannot validate | Status |
|---|---|---|---|---|---|---|
| **CelebA-Spoof** (625k images, 10k+ subjects, 10 spoof types, 43 attributes) | https://github.com/ZhangYuanhan-AI/CelebA-Spoof · Agreement: https://mmlab.ie.cuhk.edu.hk/projects/CelebA/CelebA_Spoof.html | Non-commercial, no redistribution. Download via repo links. | ✅ VERIFIED | Single-frame PAD, cross-dataset generalisation, attribute-wise analysis (lighting, environment) | Video temporal cues, active challenges, 3D masks | **USE — first dataset to download** |
| **Replay-Attack (Idiap)** (1,300 clips, 50 subjects, print/phone/tablet/video, train/devel/test split) | https://www.idiap.ch/en/dataset/replayattack · Files: https://zenodo.org/records/4593128 | Restricted — EULA; academic/professional email from same org as signatory | ✅ VERIFIED | Print + screen replay, video, calibration protocol (has official train/devel/test split) | 3D masks, modern phones/screens | **REQUEST — needed for temporal + video PAD eval** |
| **WMCA (Idiap)** (1,941 clips, 72 identities, colour/depth/IR/thermal) | https://www.idiap.ch/en/dataset/wmca · Files: https://zenodo.org/records/4580313 | Restricted — EULA; signatory must hold permanent position | ✅ VERIFIED | Multi-channel, mask/cut-out experiments. Request the RGB-only package for our pipeline. | Requires EULA signatory with permanent institutional position | **REQUEST — for secondary target validation** |
| **3DMAD (Idiap)** (76,500 frames, 17 persons, Kinect) | https://www.idiap.ch/en/dataset/3dmad · Files: https://zenodo.org/records/4068477 | Restricted — EULA | ✅ VERIFIED | 3D mask experiments | Small, low resolution, only 17 subjects | **REQUEST — for experimental 3D mask work** |
| **OULU-NPU** (4,950 videos, 55 subjects, 6 phones) | https://sites.google.com/site/aboraji/oulu-npu | Request | ❌ UNVERIFIED | Cross-phone, cross-environment generalisation | — | **REQUEST — useful for cross-device eval** |
| **SiW** (4,478 videos, 165 subjects) | http://cvlab.cse.msu.edu/siw-spoof-in-the-wild-database.html | Request | ❌ UNVERIFIED | Varied attacks, pose/expression variation, different distances | — | **REQUEST — secondary** |
| **CASIA-FASD** (600 clips) | http://www.cbsr.ia.ac.cn/english/FaceAntiSpoofDatabases.asp | Request | ❌ UNVERIFIED | Classic print/cut/replay 3-protocol benchmark | Old, low-quality capture equipment | **REQUEST — lower priority; old benchmark** |
| **MSU-MFSD** (280 videos) | Contact MSU researchers | Request | ❌ UNVERIFIED | Print + replay (laptop + Android) | Very small | **REQUEST — low priority** |

### 17.2 Deepfake / Digital Attack Datasets

These are for mitigation experiments only, not PAD claims.

| Dataset | Link | Access | Verified? | Use | Status |
|---|---|---|---|---|---|
| **FaceForensics++** | https://github.com/ondyari/FaceForensics | Request form | ❌ UNVERIFIED | Digital deepfake experiments for mitigation section of eval | **REQUEST — for deepfake mitigation eval only** |
| **Celeb-DF** | https://github.com/yuezunli/celeb-deepfakeforensics | Request | ❌ UNVERIFIED | Higher-quality deepfake evaluation | **REQUEST — secondary deepfake** |
| **UniAttackData+** (ICCV 2025, 2,875 subjects, 679k forged videos, 54 attack methods) | ICCV 2025 workshop challenge — contact Ajian Liu / Jun Wan | Institutional affiliation required | ❌ UNVERIFIED | Unified physical + digital attack evaluation | Requires institutional application | **REQUEST — aspirational; use if access granted** |

### 17.3 Bona Fide, Quality, and Fairness Datasets

| Dataset | Link | Access | Verified? | Use | Status |
|---|---|---|---|---|---|
| **ONOT** (synthetic ICAO faces, ~13k images, 3 subsets, 5 ethnic groups) | https://miatbiolab.csr.unibo.it/icao-synthetic-dataset/ | Free, CC BY-NC 4.0 | ✅ VERIFIED | Bona fide source only — print and screen-replay these yourself to make labelled attacks with no consent problem. Not an attack dataset. | **USE NOW — no consent issues** |
| **TONO** | https://miatbiolab.csr.unibo.it/tono-synthetic-dataset/ | Request | ❌ UNVERIFIED | Companion synthetic set for ICAO checks | **REQUEST** |
| **DFIC** (58,633 images + 2,706 videos, 1,016 subjects, 26 requirements annotated) | https://github.com/visteam-isr-uc/DFIC | Request — research access | ❌ UNVERIFIED | Quality gate + fairness evaluation (best real-photo set if access granted) | **REQUEST** |
| **CelebA** | http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html | Non-commercial | ✅ VERIFIED | Attribute-wise quality gate checks | **USE** |
| **LFW** | http://vis-www.cs.umass.edu/lfw/ | Open research use | ✅ VERIFIED | Genuine/impostor pairs to calibrate same-face threshold | **USE** |
| **Team-recorded set** | n/a | Team produces | n/a | **Mandatory** — ≥30 consenting subjects, all challenge actions, good/low light, glasses, face coverings, print/screen/video attacks of ONOT faces and volunteers; Monk Tone tagged | **RECORD IN P0** |

### 17.4 Models and Tools

| Item | Link | Licence | Verified? | Notes |
|---|---|---|---|---|
| **MiniFASNet / Silent-Face-Anti-Spoofing** (PyTorch source) | https://github.com/minivision-ai/Silent-Face-Anti-Spoofing | Apache-2.0 | ✅ VERIFIED | Original architecture and weights |
| **MiniFASNetV2-SE ONNX export** (~1.8 MB, INT8 ~600 KB) | https://github.com/suriAI/face-antispoof-onnx | Apache-2.0 per repo | ✅ VERIFIED (repo exists; weights not independently re-verified) | Candidate ONNX for Desktop + Android. **Verify class order empirically before trusting score direction.** |
| **OFIQ** (reference ISO/IEC 29794-5, C/C++) | https://github.com/BSI-OFIQ/OFIQ-Project · https://bsi.bund.de/dok/OFIQ-e | Check `LICENSE.md` and all dependency licences | ✅ VERIFIED (repo exists) | Optional quality gate behind `QualityGate` interface; models are a separate download; JNI/Android build is a risk |
| **OpenCV Zoo YuNet (detector) + SFace (recognizer)** | https://github.com/opencv/opencv_zoo | Apache-2.0 per model — **confirm per model file at download** | ✅ VERIFIED (repo exists) | Candidate same-face embedder |
| **Landmark model** | Not chosen yet | Must be permissive + ONNX/TFLite export | ❌ UNVERIFIED | Record in licence register at download time |
| **ISO/IEC 29794-5:2025 free preview** | https://webstore.ansi.org/preview-pages/ISO/preview_ISO+IEC+29794-5-2025.pdf | Preview only | ✅ VERIFIED (from TECHNICAL_DESIGN.md) | Reference for quality field alignment |
| **iBeta ISO 30107-3 methodology** | https://www.ibeta.com/iso-30107-3-presentation-attack-detection-confirmation-letters/ | Public | ✅ VERIFIED | Reference bar for evaluation |
| **NIST BioCTS** | nist.gov | Public | ❌ UNVERIFIED | 19794-5 conformance check of generated records |
| **EDC/pAUC quality eval tool** | https://github.com/dasec/quality-assessment-evaluation | Check licence | ❌ UNVERIFIED | Optional quality evaluation |
| **MOSIP repos** | https://github.com/mosip/registration-client · https://github.com/mosip/android-registration-client · https://github.com/mosip/mosip-mock-services | MOSIP licence | ✅ VERIFIED | Integration reference; extend MockMDS, do not rewrite |
