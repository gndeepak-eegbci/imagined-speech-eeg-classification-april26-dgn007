# Imagined Speech EEG Classification using EEGNet, Data Augmentation, and Transfer Learning

This project contains my step-by-step work on imagined speech EEG classification. Instead of trying random ideas together, I built this project slowly in multiple phases so that I could clearly understand what is actually helping and what is not.

The main goal of this work is to study why imagined speech EEG classification gives low performance and to check whether the problem can be improved by changing the model, improving the data, or using knowledge from other subjects.

I followed a simple rule throughout the project: change only one thing at a time and keep everything else the same. This helps in making fair comparisons and understanding the real impact of each method.


## Project Overview

Imagined speech EEG classification is a difficult problem because:

- EEG signals are noisy
- the dataset is small
- models easily overfit on training data
- generalization across subjects is weak

In this project, I first worked in a **subject-wise setting**, where the model is trained and tested on the same subject using 5-fold cross-validation. After that, I moved to a **transfer learning setup**, where the model first learns from other subjects and then adapts to the target subject.

The full project is divided into phases, where each phase answers one specific question:

- Does a simple EEGNet model work?
- Does adding attention improve performance?
- Does data augmentation help more than model changes?
- Which augmentation method is better?
- Can transfer learning improve performance further?
- What happens when transfer learning is combined with augmentation?



## Dataset and Preprocessing

The EEG data is loaded from `.mat` files for each subject.

Each file contains:
- EEG signal values
- modality information
- stimulus labels
- artifact indicators

The preprocessing steps are the same across all experiments:

1. Load EEG data from the subject file
2. Separate signal values from metadata
3. Keep only imagined speech trials
4. Remove trials marked as invalid or containing artifacts
5. Reshape the data into `(trials, channels, samples)`
6. Convert labels into zero-based format (0 to 10)
7. Apply stratified 5-fold cross-validation

For the main subject used in experiments:
- total valid trials: 351
- channels: 6
- samples per trial: 4096
- number of classes: 11


## Experimental Phases

### Phase 1 — Baseline EEGNet
I started with a simple EEGNet model using subject-wise 5-fold cross-validation.  
This phase was mainly to check if the pipeline is working correctly.

Result:
- The model learns training data very well
- But validation and test performance stay low  
- This clearly shows overfitting



### Phase 2 — EEGNet + Attention
Here I added a lightweight attention mechanism on top of EEGNet.

Everything else was kept the same:
- same data
- same folds
- same training pipeline

Goal:
To check if the model can focus better on important features

Result:
- Performance did not improve
- Slightly worse than baseline
- Model became more unstable

Conclusion:
Adding attention alone does not solve the problem



### Phase 3 — Gaussian Noise Augmentation
Instead of changing the model, I started improving the training data.

What I did:
- Created slightly noisy versions of training data
- Kept validation and test data untouched
- Doubled training data size

Result:
- Performance improved slightly
- Better than attention model

Conclusion:
Improving data helps more than changing architecture



### Phase 4 — Amplitude Scaling
Here I modified signal strength slightly instead of adding noise.

What this does:
- Changes amplitude of EEG signals
- Keeps pattern same but varies intensity

Result:
- Best augmentation result among all tested
- Better than Gaussian noise



### Phase 5 — Time Shifting
Here I shifted EEG signals slightly in time.

What this means:
- Signals move left or right
- Helps model not depend on exact timing

Result:
- Improvement over baseline
- But weaker than amplitude scaling



### Phase 6 — Transfer Learning
This is the most important phase.

Instead of training only on one subject:

1. Combine data from all other subjects
2. Train model on that larger dataset (pretraining)
3. Fine-tune on the target subject

Result:
- Highest mean accuracy
- Better generalization

Conclusion:
Learning from multiple subjects is more powerful than only improving data locally



### Phase 7 — Transfer Learning + Amplitude Scaling
Here I combined the best two ideas:

- Transfer learning
- Amplitude scaling

What I did:
- Pretrain on other subjects
- During fine-tuning, apply amplitude scaling

Result:
- Mean accuracy slightly lower than transfer learning alone
- But results are more stable across folds

Conclusion:
This method improves consistency even if it does not give the highest score



## Result Summary

| Method                      | Mean Accuracy | Std Deviation | Observation              |
|----------------------------|--------------|---------------|----------------------------|
| Baseline EEGNet            | 12.82%       | 2.67%         | Weak, overfitting          |
| EEGNet + Attention         | 12.53%       | 3.68%         | Did not help               |
| EEGNet + Gaussian Noise    | 14.80%       | 4.06%         | Moderate improvement       |
| EEGNet + Amplitude Scaling | 15.09%       | 4.16%         | Best augmentation          |
| EEGNet + Time Shifting     | 14.25%       | 4.06%         | Helpful but weaker         |
| Transfer Learning          | 15.66%       | 2.93%         | Best mean performance      |
| Transfer + Amplitude       | 15.38%       | 1.19%         | Most stable                |


## Key Findings

From all experiments, a very clear pattern appears:

- Changing the model (attention) did not help
- Improving training data helped more
- Transfer learning helped the most
- Combining transfer learning with augmentation improves stability

In simple words:

> Learning from more data and more subjects matters more than making the model slightly more complex.



## Important Insight

The main problem in this task is not just the model.

The real challenges are:
- small dataset
- high variability in EEG signals
- poor cross-subject generalization

So the best direction is:
- better data strategies
- cross-subject learning (transfer learning)



## Note on Notebook Organization

In this repository, I have kept the work in two formats so that it is easier to understand and also easier to run.

First, I have maintained all the experiments as separate notebooks, where each phase is written independently. This helps in clearly seeing what change was made in that particular phase and how it affected the result.

Along with that, I have also created one combined notebook that includes the full pipeline from Phase 1 to Phase 6 in a single place (combined_phase1_to_phase6_pipeline.ipynb). This notebook shows the complete flow of the work in one continuous script, which can be useful for someone who wants to follow the entire process step by step without switching between multiple files.

For Phase 7, I have kept it as a separate notebook because it builds on transfer learning and adds an additional step, so it is clearer to keep it independent.

So in simple words, the separate notebooks are useful for understanding each experiment individually, and the combined notebook is useful for seeing the overall workflow in one place.




## How to Run

1. Open the notebook in Google Colab  
2. Mount Google Drive  
3. Set dataset paths correctly  
4. Run cells step by step  

Each notebook is independent and represents one phase of the experiment.



## Folder Structure

```text
imagined-speech-eeg-classification-april26-dgn007/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── notebooks/
│   ├── 01_baseline_eegnet.ipynb
│   ├── 02_attention_eegnet.ipynb
│   ├── 03_gaussian_noise.ipynb
│   ├── 04_amplitude_scaling.ipynb
│   ├── 05_time_shifting.ipynb
│   ├── 06_transfer_learning.ipynb
│   ├── 07_transfer_amplitude.ipynb
|   ├── combined_phase1_to_phase6_pipeline.ipynb
│
├── src/
│   ├── models/
│   ├── preprocessing/
│   ├── training/
│   └── utils/
│
├── results/
│   ├── tables/
│   │   └── performance.md
│   ├── plots/
│   │   ├── accuracy_comparison.png
│   │   └── training_curves.png
│   └── logs/
│
├── images/
│   ├── architecture.png
│   └── pipeline.png
│
└── data/
    └── README.md




## Final Conclusion

This project shows a complete experimental journey.

It does not just show what worked, but also what did not work and why that matters.

The most important takeaway is:

- attention-based changes are not enough  
- data improvement helps  
- transfer learning is the strongest direction  

And even when accuracy improvement is small, reducing variability and improving stability is also a meaningful result.


