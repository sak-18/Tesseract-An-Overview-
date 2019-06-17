#   Working of Tesseract(overview) - 
An Outline

## Overview
* Introduction to Tesseract
* Architecture 
* Line detection
* Word detection
* Word recognition
* Character classification
* Improvements made in tesseract4(LSTMs)
* Results
* Limitations
* Improvements
## 1. Introduction to Tesseract
Tesseract is an open-source OCR engine that was developed at HP between 1984 and 1994.In late 2005, HP released Tesseract for open source.

## 2. Architecure
Tesseract does not have an inbuilt page layout analysis. It expects input as a preprocessed blob with polygonal text regions defined.
* The first step is a connected component analysis in which outlines of the components are stored.
* Blobs are organized into text lines, and the lines and regions are analyzed for fixed pitch or proportional text.
* Fixed pitch text is chopped immediately by character cells. Proportional text is broken into words using fuzzy spaces.
### Two pass model
* Recognition proceeds as a two pass process.
* Pass I : Attempt made to recognize each word; if satisfactory passed to adaptive classifier as training data.
* Pass II: Adaptive classifier might have learned something useful in pass I, hence another pass is made top-down
## 3.Line detection

### Line finding
* A percentile height filter removes drop-caps and vertically touching characters.
* A similar median height filter is used to eliminate noises from the preprocessed blob.
* Sorting and processing the filtered blobs by x-coordinate makes it
possible to assign blobs to a unique text line, while tracking the slope across the page, with greatly reduced danger of assigning to an incorrect text line in the presence of skew.

### Baseline Fitting
*  The base lines are fit using quadratic spline, after finding lines.
*  The baselines are fitted by partitioning the blobs into groups with a reasonably continuous displacement for the original straight baseline.
* A quadratic spline is fitted to the most populous partition, (assumed to be the baseline) by a least squares fit.
![](/img/curve-fit-baseline.png)
## 4.Word Detection
### Fixed Pitch text
* Fixed pitch and proportional text are processed differently
* Fixed pitch words are chopped down into characters
* Chopper and associator is disabled on these words
![](/img/mountain.png)
### Proportional text
* Tesseract solves most of these problems by measuring gaps in a limited vertical range between the baseline and mean line. Spaces that are close to the threshold at this stage are made fuzzy, so that a final decision can be made after word recognition.
## 5.Word Recognition
### Chopping Joined Characters
*  Candidate chop points are found from concave vertices of a polygonal approximation of the outline, and may have either another concave vertex opposite, or a line segment. It may take up to 3 pairs of chop points to successfully separate joined characters from the ASCII set.
* Chops are executed in priority order. Any chop that fails to improve the confidence of the result is undone, but not completely discarded so that the chop can be re-used later by the associator if needed.
![](/img/chop-points.png)
### Associating broken characters
* If all chops(with high confidence of separating characters) have been used, then broken characters are joined. 
* The associator searches in its segmentation graph, by maintaining a hash table of all visited states.
![](/img/broken-character.png)

## 6.Character Classification
### Static Classifier
* Early versions of tesseract used topological features
* An intermediate idea involved the use of segments of the polygonal approximation as features, but this approach is also not robust to damaged characters.
* During training, the segments of a polygonal approximation are used for features, but in recognition, features of a small, fixed length (in normalized units) are extracted from the outline and matched many-to-one against the clustered prototype features of the training data.

![](/img/feature-prototype.png)
### Adaptive Classifier
* The only significant difference between the static classifier and the adaptive classifier, apart from the training data, is that the adaptive classifier uses isotropic baseline/x-height normalization, whereas the
static classifier normalizes characters by the centroid (first moments) for position and second moments for anisotropic size normalization.

## 7.Improvements made in tesseract4
* LSTMs were integrated.
* A total of 4 modes are available(including the legacy engine mode)
### LSTMs in tesseract4
* Runs forward and backward LSTMs to predict characters in a word.
* One timestep per one pixel.
* These LSTMs take very long time to train.
## 8.Results
![](/img/result.png)

## 9. Limitations
* Uses polygonal approximation(time consuming)
## 10. Improvements to be made
* Accuracy may be improved by the addition of a hidden Markov based character n-gram model.
* Possible improvement of chopper.
## 11. References
* [Reference slides](https://github.com/tesseract-ocr/docs/tree/master/das_tutorial2016)
* [An Overview of Tesseract-Ray Smith](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=2ahUKEwjXgbb8mujiAhXHXCsKHQBOAiAQFjAAegQIAhAC&url=https%3A%2F%2Fresearch.google.com%2Fpubs%2Farchive%2F33418.pdf&usg=AOvVaw3MUJfEGXPwnQHIVc9SrO4E)
