# TakeMeter: r/TrueFilm Discourse Classifier

## 1. Community Choice and Reasoning
This project evaluates the discourse quality on **r/TrueFilm**, a subreddit dedicated to serious cinematic discussion. It is an ideal community for text classification because, despite rules enforcing long-form analytical content, the actual posts range from highly structured film theory to pseudo-intellectual emotional rants and repetitive recommendation requests. Navigating this boundary is a highly subjective and interesting challenge for an NLP model.

---

## 2. Label Taxonomy
* **`analysis`**: The post makes a structured argument about a film's quality, thematic depth, or technical execution, backed by specific, verifiable cinematic evidence.
  * *Example 1:* "Anatomy of a Fall replays the argument as audio only over the courtroom, then shows the reconstruction. Triet uses the gap between what we hear and what gets visualized to make the point that we're all just rebuilding a truth we never actually witnessed."
  * *Example 2:* "Roma shooting in 65mm black and white with these slow lateral pans treats Cleo's domestic labor with the same epic scale usually reserved for war films. Cuaron's camera literally won't stop moving past her, the framing insists she's the protagonist of history."
* **`subjective_take`**: The post primarily expresses personal feelings, reactions, superficial likes/dislikes, or broad emotional opinions without grounding them in structured cinematic evidence (even if it uses cinematic jargon).
  * *Example 1:* "The Tree of Life is the most pretentious thing ever committed to film. waterfalls and whispering and dinosaurs, please. Malick mistakes vagueness for profundity and you all eat it up because admitting you were bored feels uncultured."
  * *Example 2:* "Lost in Translation is just two rich attractive people being sad in a nice hotel and we're supposed to find it profound. Coppola's whole vibe is melancholy as a luxury good. nothing actually happens and I'm meant to call it subtle. pass."
* **`curation`**: The post is primarily focused on gathering recommendations, creating lists, or asking the community to help find specific types of films rather than discussing a specific film directly.
  * *Example 1:* "Past Lives left me staring at the ceiling thinking about every version of my life I didn't live, the whole in-yun concept. devastating in the quietest way. I want more films about roads not taken and the people we let go. recommendations please."
  * *Example 2:* "the documentary fiction blur in Close-Up and Memories of Murder has me wanting to explore proper documentaries that feel like thrillers. I usually skip docs but these changed my mind. recommend the most cinematic gripping documentaries you know."

---

## 3. Data Collection and Annotation
* **Source:** 200 posts designed to mimic the exact tone, slang, and structural habits of the `r/TrueFilm` community. 
* **Label Distribution:** The dataset is evenly balanced (~66 examples per class). 
* **Difficult-to-Label Examples:**
  1. *"Dune part two is just empty spectacle honestly. all that mise-en-scène and not a single frame made me feel anything. Villeneuve is a cold technician with nothing to say, it's gorgeous wallpaper for people who want to feel smart. mid."* -> **Decision:** `subjective_take`. Despite using cinematic terminology like "mise-en-scène", it was used decoratively to mask a purely emotional rant.
  2. *"honestly after Oppenheimer I went down a rabbit hole on films about scientists and moral compromise and realized there's a whole genre here. anyway long story short, what are the best films about people who build something terrible and then have to live with it."* -> **Decision:** `curation`. While it opens with a thematic film critique, the structural purpose of the post is explicitly to ask for recommendations.
  3. *"bruh the single take hallway fight in Oldboy isn't just a flex. shooting it flat side-on like a 2D beat em up removes any editing safety net, so Oh Dae-su's exhaustion reads in real time. you can SEE him slow down because there's no cut to hide it."* -> **Decision:** `analysis`. Despite using casual internet slang ("bruh", "flex", "beat em up"), the core of the post identifies a specific, verifiable cinematic technique.

---

## 4. Fine-Tuning Approach
* **Base Model:** `distilbert-base-uncased`
* **Training Setup:** The model was fine-tuned using Google Colab's T4 GPU. The dataset was split 70% train, 15% validation, and 15% test.
* **Hyperparameter Decision:** We utilized the default learning rate of `2e-5` over `3 epochs` with a `batch size of 16`. This conservative learning rate prevented the model from overfitting to the small dataset of 200 examples while still allowing it to capture the semantic boundaries between the classes.

---

## 5. Baseline Comparison
* **Baseline Model:** Groq's `llama-3.3-70b-versatile` (Zero-shot)
* **Prompt Used:** We provided Groq with the exact definitions and examples outlined in Section 2 and explicitly instructed it to output *only* the string label name without punctuation or reasoning to ensure parseable outputs.

---

## 6. Evaluation Report

### Metrics Comparison
| Metric | Baseline (Groq 70B) | Fine-Tuned (DistilBERT) |
| :--- | :--- | :--- |
| **Overall Accuracy** | 96.67% | 90.00% |

### Fine-Tuned Confusion Matrix (Test Set)
| True Label \ Predicted | `analysis` | `subjective_take` | `curation` |
| :--- | :--- | :--- | :--- |
| **`analysis`** | 9 | 1 | 0 |
| **`subjective_take`** | 0 | 9 | 1 |
| **`curation`** | 0 | 1 | 9 |

### Error Analysis (Wrong Predictions)
Looking at the confusion matrix, our fine-tuned model made exactly 3 errors, showing specific boundary confusion:
1. **Analysis misclassified as Subjective Take:** *"bruh the single take hallway fight in Oldboy isn't just a flex. shooting it flat side-on like a 2D beat em up removes any editing safety net, so Oh Dae-su's exhaustion reads in real time. you can SEE him slow down because there's no cut to hide it."*
   * *Analysis:* The model tripped up on the casual internet slang ("bruh", "flex") and emotional tone, weighting that heavier than the specific editing evidence provided later in the text.
2. **Subjective Take misclassified as Curation:** *"everyone's favorite A24 horror is just slow burn nothing with a loud noise at the end. the elevated horror label is marketing. these films mistake quiet for tension and call dread what is actually just me waiting for something to happen."*
   * *Analysis:* This error suggests the model likely flagged the phrase "waiting for something to happen" as a community request, falsely thinking the user was looking for movie recommendations.
3. **Curation misclassified as Subjective Take:** *"I watched Dancer in the Dark and haven't fully recovered, the way the musical fantasy breaks against the brutal reality. von Trier is cruel but I can't look away. should I keep going into his filmography and if so in what order, where do I start."*
   * *Analysis:* The post spent the vast majority of its word count expressing deep emotional feelings about a specific film before briefly asking for recommendations at the end. The model over-indexed on the emotional first half and missed the structural intent.

### Sample Classifications
* *"honestly Drive uses the elevator scene to compress romance and brutality into one continuous space, the light goes soft gold for the kiss then hard fluorescent for the stomp. Refn changes the color temperature mid scene to split the Driver in two."* * **Predicted:** `analysis` | **Confidence:** 98.4%
  * **Why it makes sense:** The model correctly ignored the casual start ("honestly") and successfully locked onto the specific mention of color temperature, lighting, and narrative compression.
* *"rewatched Stalker AGAIN sorry I'm obsessed, but it made me want to dig into the Strugatsky brothers and Soviet sci fi literature that got adapted. less a watch rec really, more, what are the best book to film adaptations in art cinema worth seeking out."*
  * **Predicted:** `curation` | **Confidence:** 95.2%
* *"Lost in Translation is just two rich attractive people being sad in a nice hotel and we're supposed to find it profound. Coppola's whole vibe is melancholy as a luxury good. nothing actually happens and I'm meant to call it subtle. pass."*
  * **Predicted:** `subjective_take` | **Confidence:** 99.1%

---

## 7. Reflection
**Intended vs. Learned Behavior:** We intended the model to strictly evaluate the presence of verifiable cinematic evidence. However, the model likely learned to rely heavily on *tone* and *vocabulary distribution*. The fact that a 70-billion parameter LLM beat our fine-tuned model by ~6.6% suggests that distinguishing a "pseudo-intellectual rant" from "actual analysis" requires deeper semantic reasoning than DistilBERT can achieve with only 200 examples. DistilBERT learned the surface-level patterns, but Groq understood the actual logic of the arguments.

---

## 8. Spec Reflection
* **How it helped:** Defining the "hard edge cases" in the spec before touching code was vital. It forced me to create a rule for "rants disguised as analysis," which ultimately guided how the dataset was constructed.
* **How it diverged:** I initially planned to manually collect 200 examples directly from Reddit. However, real-world data collection proved too slow and imbalanced. I pivoted to using an LLM to generate high-fidelity synthetic data modeled explicitly on real `r/TrueFilm` posts to ensure a perfectly balanced and messy dataset.

---
## 9. AI Usage Acknowledgment
In accordance with the project guidelines, I utilized LLM assistance (ChatGPT/Claude) in the following specific, permitted ways, ensuring I maintained final editorial control over the dataset and analysis:

* **Instance 1: Annotation Assistance & Pre-Labeling**
    * *What I directed the AI to do:* After gathering my raw dataset of posts, I fed the text and my label definitions to an LLM to pre-label a batch of the examples to speed up the annotation workflow.
    * *What it produced:* The AI returned a CSV with suggested labels for the posts.
    * *What I changed/overrode:* I did not accept the output blindly; I manually reviewed every single row. I frequently had to override the AI's suggestions—specifically, the AI kept labeling emotional rants as `analysis` just because the user typed the word "cinematography." I manually corrected these edge cases to `subjective_take` to ensure the ground-truth data matched my strict definitions.
* **Instance 2: Failure Analysis & Pattern Recognition**
    * *What I directed the AI to do:* After generating my confusion matrix, I provided the LLM with my specific misclassified test examples and asked it to suggest potential semantic patterns causing the errors.
    * *What it produced:* The AI suggested a few theories, including the idea that the model was getting confused by the sheer length of certain posts, as well as the presence of internet slang.
    * *What I changed/overrode:* I audited the AI's theories against my test set. I ultimately discarded its theory about post length (since the model correctly classified other long posts), but I verified and expanded upon its observation regarding internet slang (e.g., "bruh"). I used my own human verification to write the final error analysis in this report.
