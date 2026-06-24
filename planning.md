# TakeMeter Project Planning Spec: r/TrueFilm

## 1. Community
The chosen community is **r/TrueFilm**, a public Reddit forum dedicated to in-depth, structured discussion of film. It is an ideal fit for a text classification task because its discourse ranges drastically in analytical quality. While the subreddit explicitly enforces rules requiring long-form, thoughtful posts, the actual content varies from formal cinematic analysis to highly emotional personal rants or repetitive recommendation requests. This creates an interesting and highly subjective boundary for a text classifier to navigate.

---

## 2. Label Taxonomy
We define three mutually exclusive and exhaustive labels to categorize the quality and type of discourse:

* **`analysis`**: The post makes a structured argument about a film's quality, thematic depth, or technical execution, backed by specific, verifiable cinematic evidence (such as editing techniques, cinematography, narrative structure, or historical context).
  * *Example 1:* "The use of deep focus in Citizen Kane isn't just a technical gimmick; it actively reflects Kane's psychological isolation by keeping his massive, empty estate constantly in sharp view behind him."
  * *Example 2:* "A look at the pacing of Tarkovsky's Stalker reveals how long takes are mathematically structured to induce a meditative state, forcing the viewer to experience time identically to the characters."
* **`subjective_take`**: The post primarily expresses personal feelings, reactions, superficial likes/dislikes, or broad emotional opinions without grounding them in structured cinematic evidence.
  * *Example 1:* "I tried watching Vertigo last night and I honestly just didn't vibe with it. It felt way too slow and dated, and I couldn't connect with any of the characters at all."
  * *Example 2:* "Interstellar is an absolute emotional masterpiece. The soundtrack alone makes me cry every single time, and it is easily Nolan's best movie by a mile."
* **`curation`**: The post is primarily focused on gathering recommendations, creating lists, or asking the community to help find specific types of films rather than discussing a specific film directly.
  * *Example 1:* "What are some essential French New Wave films for someone who has only seen Breathless? Looking to expand my watchlist this weekend."
  * *Example 2:* "Let's compile a definitive list of the best psychological thrillers from the 1990s that don't rely on cheap jump scares."

---

## 3. Hard Edge Cases
* **The Borderline Case:** A post that is highly emotional and opinionated (suggesting `subjective_take`) but drops a single piece of specific technical jargon or a scene reference to appear credible.
* **Decision Rule:** If the technical detail or scene reference is actively integrated into a structured, logical argument that supports the main thesis, classify it as `analysis`. If the detail is treated as a decorative throwaway line or an unelaborated buzzword ("the cinematography was good") inside a purely emotional rant, classify it as `subjective_take`.

---

## 4. Data Collection Plan
* **Source:** Public posts and top-level comments from the `r/TrueFilm` subreddit.
* **Volume:** At least 200 total unique examples will be collected manually into a single CSV file.
* **Target Distribution:** We will target an even distribution of roughly 65–70 examples per label.
* **Handling Imbalance:** If any single label exceeds 70% of the dataset, or if a label like `curation` is underrepresented during random sampling, we will explicitly search for underrepresented post types using relevant keywords (e.g., "recommendation", "looking for", "watchlist") until every label represents at least 20% of the complete dataset.

---

## 5. Evaluation Metrics
We will evaluate both the fine-tuned model and the zero-shot baseline using the following metrics:
* **Overall Accuracy:** To measure the overall percentage of correct predictions across the entire test set.
* **Per-Class Precision:** To see when the model predicts a class (e.g., `analysis`), how often it is actually correct. This ensures the model isn't over-predicting a specific category.
* **Per-Class Recall:** To evaluate how many of the actual examples of a class the model successfully captures, highlighting if it is being too conservative.
* **Per-Class F1-Score:** The harmonic mean of precision and recall. Because text classification on subjective internet discourse is prone to class boundary confusion, the F1-score will serve as our primary indicator of class-specific health.
* **Confusion Matrix:** To visually map exactly where boundaries are breaking down (e.g., if `subjective_take` is consistently leaking into `analysis`).

---

## 6. Definition of Success
For this classifier to be considered useful for real-world automated community moderation or text filtering, it must achieve:
* An **Overall Accuracy of $\ge$ 75%** on the test set.
* A **Per-Class F1-Score of $\ge$ 0.70** for all three classes.
* It must definitively outperform the Groq `llama-3.3-70b-versatile` zero-shot baseline in overall accuracy by at least 5%.

---

## 7. AI Tool Plan
* **Label Stress-Testing:** We will use an LLM to generate borderline cases based on these definitions to see if our decision rules hold up before human annotation begins.
* **Annotation Assistance:** We will not use an LLM to pre-label the dataset. To ensure pristine, high-fidelity ground truth data that strictly follows our custom decision rules, all 200+ examples will be manually read, verified, and labeled by the human annotator.
* **Failure Analysis:** After evaluation, the text of all misclassified examples along with their true and predicted labels will be passed to an LLM to automatically cluster errors and identify systematic semantic patterns (e.g., short lengths or heavy sarcasm). These clusters will then be manually audited for final reporting.
