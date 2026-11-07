# BurVision Dataset Builder 

An open-source synthetic data generation and document rendering pipeline for building high-resolution, layout-aware Burmese (Myanmar) OCR datasets with image and text annotations.

**BurVision** automatically harvests **140+ free & open Myanmar Unicode fonts** across 4 major open-source font repositories ([AungMyoKyaw/Myanmar-Unicode-Fonts](https://github.com/AungMyoKyaw/Myanmar-Unicode-Fonts), [Google Fonts](https://github.com/google/fonts), [lecramyajiv/fonts-myanmar-extra](https://github.com/lecramyajiv/fonts-myanmar-extra), and [saturngod/myanmar-unicode-fonts](https://github.com/saturngod/myanmar-unicode-fonts)), combines **intra-page header + body font pairings**, formats long Burmese stories into structured A4 book pages with `#H1`, `#H2`, `#H3` hierarchy and inline styling, and renders them via **Headless Chromium/Edge (`mym2` HarfBuzz + DirectWrite)** for 100% authentic Burmese glyph shaping.

## 1. Key Architecture & Dataset Specifications

| Feature | Specification |
| :--- | :--- |
| **Total Myanmar Unicode Fonts** | **140+ fonts** harvested across 4 repositories (`AungMyoKyaw/Myanmar-Unicode-Fonts`, `Google Fonts` full weights 100–900, `lecramyajiv/fonts-myanmar-extra`, `saturngod/myanmar-unicode-fonts`) |
| **Intra-Page Font Pairing** | **Dual-Font Book Typography** (pages pair a dedicated display/bold Header Font for `#H1`/`#H2`/`#H3` with a distinct Body Font) |
| **Ground Truth Labels** | **Rich Structural Markdown** (`#`, `##`, `###` headings, `**bold**`, `*italic*`, `<u>underline</u>`, `“...”` quotes) |
| **Source Corpus** | **Synthetic Literary Corpus** generated via **Gemini 3.5 Flash-Lite** across 8 cultural, historical, and modern themes |
| **Shaping & Encoding Purity** | **100% Verified Pure Unicode 5.1+ (`NFC`)** (`paraencoder` auto-conversion + strict exclusion of non-Unicode Zawgyi/Win fonts + Browser `mym2` HarfBuzz shaping) |
| **Realistic Scan Augmentation** | **Warm Book Paper + Spine Shadow + Micro-Rotation (`±0.5°`) + Sensor Grain** |

## 2. Synthetic Literary Corpus Generation (Gemini 3.5 Flash-Lite)

This repository shares the dataset-generation workflow, not a bundled text corpus or generated dataset. The working folder `burmese_ocr_project/` is intentionally excluded from Git, so each person using the notebook supplies their own UTF-8 Burmese text files. The following optional prompt can be used with Gemini 3.5 Flash-Lite (or another text-generation tool) to create source stories.

### Master Prompt

```text
System / Master Instruction:
You are a master native Burmese literary writer. Output ONLY raw Burmese text in standard Unicode (NFC). Do not include any English intros, conversational text, or markdown symbols at the start of lines.
Write an ultra-long, deeply detailed Burmese story (1,200 to 2,000+ words—make it as long as possible). Rotate through or combine these themes:
၁။ မြန်မာ့သမိုင်းနောက်ခံ ပုဂံခေတ် အဖြစ်အပျက်တစ်ခုအကြောင်း ဝတ္ထုတို
၂။ ကျေးလက်ရွာကလေးတစ်ရွာ၏ ရိုးရာဓလေ့နှင့် သဘာဝပတ်ဝန်းကျင်အကြောင်း
၃။ မြန်မာရိုးရာ ဒဏ္ဍာရီ ပုံပြင်ဆန်ဆန် ဇာတ်လမ်းတစ်ပုဒ်
၄။ မြို့ပြလူနေမှုဘဝ၊ မိသားစုမေတ္တာနှင့် ကြိုးစားအားထုတ်မှုအကြောင်း ဝတ္ထုတို
၅။ ဆန်းကြယ်သော တောတွင်းခရီးသွား စွန့်စားခန်းပုံပြင်
၆။ ပညာရေး၊ စာပေလေ့လာလိုက်စားမှုနှင့် ဆရာတပည့် သံယောဇဉ်အကြောင်း
၇။ ကုန်သည်ကြီးများ၏ ရောင်းဝယ်ဖောက်ကားမှုနှင့် စီးပွားရေးအခြေအနေအကြောင်း
၈။ ဗုဒ္ဓဝင် သို့မဟုတ် ငါးရာ့ငါးဆယ် ဇာတ်နိပါတ်တော် လာ အကြောင်းအရာတစ်ခု
Strict Formatting & Style Rules:
- Trailing Heading Tags Only: Put #H1, #H2, and #H3 at the very end of the heading lines:ခေါင်းစဉ် - [Title] #H1[Major Section Title] #H2[Sub-section Scene Title] #H3
- Structure: Include 1 #H1 title, at least 3 #H2 major sections, and at least 2 #H3 sub-sections under each #H2 section. Write 2 to 4 long, rich paragraphs under every single heading.
- Language: Use rich Burmese vocabulary, stacked consonants (ပါဠိဆင့်), Burmese numbers (၀-၉), vivid atmospheric descriptions, and natural dialogue with 100% correct spelling (စာလုံးပေါင်းသတ်ပုံ အမှန်).
```

## 3. Why Native Browser HarfBuzz (`mym2`) Shaping Matters

In standard Myanmar Unicode (`NFC`), characters are stored in **logical phonetic order**:

* The word `တောင်` is stored in memory as `တ` + `ေ` (`U+1031`) + `ာ` + `င` + `်`.
* The word `ပြည်` is stored in memory as `ပ` + `ြ` (`U+103C`) + `ည` + `်`.

Lightweight PDF/image libraries (such as PyMuPDF's `insert_htmlbox` or Pillow without `libraqm`/`fribidi.dll`) draw glyphs in raw memory order without running the OpenType **`mym2` (Myanmar v2)** shaping state machine—resulting in unshaped text where `ေ` and `ြ` appear on the wrong side of the consonant (`တေောင်`, `ပည်`).

**BurVision solves this permanently** by serving all 140+ Unicode fonts over a local background HTTP thread to a single persistent **Headless Chrome / Microsoft Edge** instance via Selenium. The browser's native **HarfBuzz + DirectWrite** engine executes full OpenType `GSUB`/`GPOS` shaping at **~0.1 seconds per page** with zero external C/GTK installation steps on Windows, macOS, or Linux.

## 4. All 140+ Harvested Myanmar Unicode Fonts (4 Repositories)

The dataset builder automatically downloads, deduplicates, and verifies 140+ distinct Myanmar Unicode `.ttf`/`.otf` font files across four open-source collections while strictly blocking legacy non-Unicode encodings (`Zawgyi-One`, `WININNWA`, `Padauk-Zawgyi`):

1. **[AungMyoKyaw/Myanmar-Unicode-Fonts](https://github.com/AungMyoKyaw/Myanmar-Unicode-Fonts) (78 Pure Unicode Fonts):**
   * *Official & Book Standards:* `pyidaungsu` (1.2, 1.3, regular), `mm3-multi-os`, `mm3_05-Jan-2011`, `mm3h`, `mm3sh`, `MM3_Paoh`, `Padauk`, `Padauk-bold`, `padauk-book`, `padauk-bookbold`, `PadaukGrandPro_v2`, `Padauk_Kyaungchikote`, `PadaukSagar`, `PadaukSgaw` (Regular, Bold, Book, BookBold), `MyanmarText (mmrtext, mmrtextb)`, `MyanmarTextMelano`, `TharLon`, `ThanLwin`, `WinUniInnwa`, `Parabaik20080421`, `Parabaik_UTN11-3`, `Yunghkio`, `MON3Anonta1`, `PangLong_2011feb7`, `myanmarcensus`.
   * *Stylized, Editorial & Display Fonts:* `Cherry_Unicode`, `Jasmine_Unicode`, `KT_02`, `KT_03`, `MasterpieceUniRound`, `MyanmarAngoun`, `MyanmarBlack`, `MyanmarGantGaw`, `MyanmarKhway`, `MyanmarKuttar`, `MyanmarNayone`, `MyanmarNjaun`, `MyanmarPaOhOne`, `MIUI_Unicode`, `MyanmarPauklay`, `MyanmarPhetsot`, `MyanmarPixel`, `MyanmarPixelSmooth`, `MyanmarPonenyet`, `Myanmar_Punk`, `Myanmar_Rosemarry`, `MyanmarSabae`, `MyanmarSagar`, `MyanmarSansPro`, `MyanmarSquare`, `MyanmarSquareLight`, `MyanmarTagu`, `MyanmarTaungyi`, `MyanmarThuriya`, `MyanmarThuriya_volt`, `MyanmarYinmar`, `NK_Grand`, `NK_SSmart2`, `NK_SSmart3`, `NK_SSmart4`, `NamKhoneUnicode`, `Namkhon_Web_pro`, `PaohDragonApp`, `Samsung_Myanmar`, `Shan_Unicode`, `YoeYar-One` (Regular, Bold, Light).
2. **[Google Fonts (`google/fonts`)](https://github.com/google/fonts) (16 Full-Spectrum Weight Fonts):**
   * Complete weight spectrum (**Thin 100, ExtraLight 200, Light 300, Regular 400, Medium 500, SemiBold 600, Bold 700, ExtraBold 800, Black 900**) for both `NotoSansMyanmar` and `NotoSerifMyanmar`.
3. **[lecramyajiv/fonts-myanmar-extra](https://github.com/lecramyajiv/fonts-myanmar-extra) (20+ Masterpiece & Debian/Slackware Extra Myanmar Fonts):**
   * Includes `Masterpiece Spring Rev`, `Masterpiece Lakwel`, `Masterpiece CTL`, `Masterpiece Stadium`, `Masterpiece TawWin`, `Masterpiece UniHand`, `Masterpiece UniSans`, `Masterpiece UniSerif`, `Masterpiece UniType`, `Masterpiece YayChanzin`, `Myanmar Chatu`, `Myanmar ChatuLight`, `Myanmar HeadOne`, `Myanmar Khyay`, `Myanmar Sanpya`, `Myanmar Waso`, and `Kayphodu`.
4. **[saturngod/myanmar-unicode-fonts](https://github.com/saturngod/myanmar-unicode-fonts) (30+ Community Designer Burmese Unicode Fonts):**
   * Harvests additional community-designed Myanmar Unicode display, handwriting, and modern book fonts curated at [font.saturngod.net](https://font.saturngod.net/).

## 5. Repository Structure

```text
burvision-dataset/
├── readme.md                         # Workflow documentation and optional text-generation prompt
├── burmese_ocr_dataset_builder.ipynb # Local dataset generator
└── burmese_ocr_project/              # Created locally; excluded from Git
   ├── raw_texts/                    # Add your UTF-8 .txt input files here
   ├── fonts/                        # Downloaded fonts
   └── dataset_output/               # Generated images, train.jsonl, and val.jsonl
```

The shared repository contains the notebook and this README. When you run the notebook, it creates `burmese_ocr_project/` locally. That folder is ignored by Git and is not shared; put your own `.txt` files in `burmese_ocr_project/raw_texts/` before running the dataset-generation cells.

## 6. Build a Dataset from Your Own Text

1. Clone or download this repository, then run the notebook's setup cells. They create `burmese_ocr_project/raw_texts/` locally; put one or more of your own UTF-8 `.txt` files there. Heading tags such as `#H1`, `#H2`, and `#H3` are optional.
2. Open `burmese_ocr_dataset_builder.ipynb` and run the cells from top to bottom. The font harvester needs an internet connection, and rendering requires Chrome or Microsoft Edge.
3. Find the generated page images and `train.jsonl` / `val.jsonl` files in `burmese_ocr_project/dataset_output/`.

## 7. Exporting & Publishing the Dataset

Run **Block 8** to create `burmese_ocr_poc.zip` locally from the generated dataset. The notebook does not require credentials or upload data to GitHub, Hugging Face, or Kaggle; publish the generated files or archive manually to the platform you choose.

## 8. References & Acknowledgments

* **Font Collections:** [AungMyoKyaw/Myanmar-Unicode-Fonts](https://github.com/AungMyoKyaw/Myanmar-Unicode-Fonts), [Google Fonts](https://github.com/google/fonts), [lecramyajiv/fonts-myanmar-extra](https://github.com/lecramyajiv/fonts-myanmar-extra), and [saturngod/myanmar-unicode-fonts](https://github.com/saturngod/myanmar-unicode-fonts).
