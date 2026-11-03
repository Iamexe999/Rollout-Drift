# When a frozen frame scores 99.91% pixel agreement

How far can a prediction miss a moving object before the pixel score notices? In a controlled moving-ball scene, the score stopped changing once the predicted and true balls no longer overlapped. Between four and sixteen frames ahead, the position error grew from 8 to 32 pixels. The percentage of wrong pixels stayed exactly the same.

This matters when using a world model to plan an action such as catching a ball. An eight-pixel miss and a 32-pixel miss imply different corrections. An evaluation metric should tell us when those corrections are growing.

## Experiment

I compared two predictors on 32 synthetic clips. Each showed a white disk of radius three pixels moving horizontally by two pixels per frame on a black background. A fixed random seed varied its starting position and direction. Both predictors had access to the same two observed frames. The freeze baseline repeated the last one. The velocity predictor measured the displacement between the two observed centers and shifted the disk by that amount for each future step. The predictions ran without further observations. Every disk stayed inside the frame.

I scored each forecast two ways. Pixel mismatch is the percentage of pixels that differ from the true frame. Center error is the Euclidean distance between the predicted and true disk centers. To isolate the effect of background area, I scored each scene at 128 × 128 and again with a black border that increased the canvas to 256 × 256. The disk, speed and displacement stayed identical.

## Results

Mean errors for the frozen-frame baseline across 32 clips:

| Frames ahead | Center error (pixels, both sizes) | Wrong pixels at 128 × 128 | Wrong pixels at 256 × 256 |
|---:|---:|---:|---:|
| 1 | 2 | 0.1465% | 0.0366% |
| 4 | 8 | 0.3540% | 0.0885% |
| 8 | 16 | 0.3540% | 0.0885% |
| 16 | 32 | 0.3540% | 0.0885% |

The velocity predictor had zero error under both metrics at every horizon. Its motion assumption matches the simulator exactly, so it serves as a check on the scoring code. Both metrics ranked the predictors in the same order. They disagreed about how much worse the frozen prediction became over time.

Each disk occupies 29 pixels. Once the two disks separate, the frozen forecast puts 29 white pixels where the background should be and leaves 29 black pixels where the ball should be. Moving the disks farther apart still gives 58 wrong pixels. Padding the frame quadruples its area and divides the reported error by four. Across these clips, changing the initial position and direction leaves that geometry intact.

## What would change the conclusion?

The result establishes a failure mode for whole-frame mismatch in a sparse binary scene. It leaves open how often the same issue matters for learned models or natural video. With another week, I would test a learned predictor on held-out bouncing-ball clips, vary ball size, and score both before and after wall impacts. Within each size and horizon, I would check whether pixel error reliably identifies the forecasts with the largest center errors. Consistent agreement across those settings would narrow my practical concern to scenes like this one.

I would like to work on evaluating whether predicted video preserves the object motion that a downstream decision depends on. My computer vision experience makes tracking a natural starting point. A next step would be to test which metric best predicts a missed interception.

[Run the experiment or inspect the full output](rollout_drift.ipynb). The notebook requires Python 3 and NumPy. Run its single code cell to reproduce the table.

**AI use disclosure.** I used ChatGPT to draft the analysis.
