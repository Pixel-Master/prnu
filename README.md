# PRNU Toolkit & AI Predictor

A Python library for Photo-Response Non-Uniformity (PRNU) noise extraction, camera identification through fingerprint correlation and a ResNet-18-based AI image detector.

## Links
Refer to the [interactive website](https://Pixel-Master.github.io/prnu) as a usage example and the [repository](https://gihtub.com/Pixel-Master/prnu)


## Installation

```
pip install prnu
```


## Usage Examples

### 1. Extract visible noise

Extract PRNU noise from an image and scale subtle variance into a viewable 8-bit PNG image:

```python
import prnu

# Initialize extractor with an image file path
extractor = prnu.NoiseExtractor("image.png")

# Extract PRNU multiplicative noise residual
extractor.extract_noise()

# Make the noise visible
extractor.shift_to_visual()

# Save visual representation to disk
extractor.write_to_file("visual_noise.png")

```

### 2. Camera identification

Average the noise patterns from training photos taken by a single camera, compute an average reference fingerprint, and then test an unknown image:

```python
import os
import prnu

training_dir = "/path/to/camera_samples"
test_image_path = "/path/to/test_image.jpg"

# 1. Collect noise from training photos
avg = prnu.AverageNoise()
for file_name in os.listdir(training_dir):
    file_path = os.path.join(training_dir, file_name)
    
    # Extract noise without zero-meaning during aggregation
    extractor = prnu.NoiseExtractor(file_path)
    extractor.extract_noise(disable_zero_meaning=True)
    avg.add_image_data(extractor)

# Compute final reference fingerprint
avg.analyse()

# 2. Extract noise from test image
test_extractor = prnu.NoiseExtractor(test_image_path)
test_extractor.extract_noise()

# 3. Correlate test image against reference profile
comparison = prnu.Compare(test_extractor, avg)
score = comparison.correlate()

print(f"Correlation Score: {score:.5f}")
if score > 0.008:
    print("The test image likely originated from the target camera sensor.")

```
Generally with a correlation score of 0.008 or higher a photo can be considered as being from the same camera

### 3. AI Image Detection

Load the attached trained ResNet-18 model to predict the likelihood that a give image is AI-generated:

```python
import prnu.ai_predictor as ai_predictor

# Initialize model instance
model = ai_predictor.Model() # this model is saved in your python installation

model.load():
# Returns scaled AI probability between 0 and 1
probability = model.predict("sample_image.png", optimal_probability=0.97) # For the included model 0.97 is the optimal threshold
print(f"AI Probability: {probability * 100:.2f}%")

```


## License

This software is licensed under the GPLv3 License.