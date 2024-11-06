import org.opencv.core.*;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;
import org.opencv.videoio.VideoCapture;

public class RealTimeImageComparison {

    static {
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
    }

    public static void main(String[] args) {
        // Path to reference image
        String referenceImagePath = "reference_image.png";
        Mat referenceImage = Imgcodecs.imread(referenceImagePath);

        if (referenceImage.empty()) {
            System.out.println("Reference image not found.");
            return;
        }

        // Coordinates of the known missing objects
        int[][] missingObjects = {
            {100, 150, 50, 50},  // x, y, width, height
            {200, 250, 30, 30}
        };

        // Open camera
        VideoCapture camera = new VideoCapture(0);
        if (!camera.isOpened()) {
            System.out.println("Error: Camera could not be opened.");
            return;
        }

        // Capture a frame
        Mat liveImage = new Mat();
        if (camera.read(liveImage)) {
            // Ensure live and reference images have the same dimensions
            if (liveImage.size().equals(referenceImage.size())) {
                // Create a result image to store differences
                Mat resultImage = liveImage.clone();

                // Process each missing object region
                for (int[] region : missingObjects) {
                    int x = region[0];
                    int y = region[1];
                    int width = region[2];
                    int height = region[3];

                    // Extract the regions from both images
                    Mat referenceRegion = referenceImage.submat(new Rect(x, y, width, height));
                    Mat liveRegion = liveImage.submat(new Rect(x, y, width, height));

                    // Calculate absolute difference between regions
                    Mat diff = new Mat();
                    Core.absdiff(referenceRegion, liveRegion, diff);

                    // Highlight the region in red if there's a difference
                    Scalar meanDiff = Core.mean(diff);
                    if (meanDiff.val[0] > 50) { // Threshold for difference
                        Imgproc.rectangle(resultImage, new Point(x, y), new Point(x + width, y + height), new Scalar(0, 0, 255), 2);
                    }
                }

                // Save the result image with highlighted differences
                Imgcodecs.imwrite("real_time_difference.png", resultImage);
                System.out.println("Difference image saved as real_time_difference.png");

            } else {
                System.out.println("Live image and reference image dimensions do not match.");
            }
        } else {
            System.out.println("Error: Could not capture image from camera.");
        }

        camera.release();
    }
}
