# Let's Buy - Virtual Try-On Shopping Mini Program

This project is an AI-powered virtual try-on shopping system built around OOTDiffusion. It includes a WeChat mini-program front end and a Python-based back-end service for generating try-on results.

Users can upload a person image and a garment image to preview outfit combinations before purchase.

## Project Overview

The system combines shopping functionality with AI-based virtual fitting to help users preview clothing styles in advance.

### Main Features

- Virtual try-on for upper-body and full-body clothing
- Product browsing and shopping flow
- Cart and order management
- User profile and account section
- Gradio-based web interface for testing and demonstration

## Project Structure

```text
.
├── miniprogram/              # WeChat mini-program front end
│   ├── pages/                # Page directory
│   │   ├── sy/               # Home page
│   │   ├── xnsy/             # Virtual try-on page
│   │   ├── cart/             # Shopping cart
│   │   ├── dd/               # Orders
│   │   └── me/               # User center
│   ├── images/               # Image resources
│   └── utils/                # Utility functions
│
└── tryon/1/                  # Virtual try-on back-end service
    ├── ootd/                 # OOTDiffusion core model
    │   ├── inference_ootd_hd.py
    │   ├── inference_ootd_dc.py
    │   └── pipelines_ootd/
    ├── preprocess/           # Preprocessing modules
    │   ├── openpose/
    │   └── humanparsing/
    └── run/
        ├── gradio_ootd.py    # Gradio web UI
        └── run_ootd.py       # CLI script
```

## Technology Stack

### Back End

- PyTorch
- OOTDiffusion
- OpenPose
- Human Parsing
- Gradio
- PIL and OpenCV

### Front End

- Native WeChat mini-program framework
- WeChat Developer Tools

## Functional Highlights

### Virtual Try-On Modes

1. Half-body mode
   - Focused on upper-body try-on
   - High-resolution output
   - Fast generation

2. Full-body mode
   - Supports tops, bottoms, and dresses
   - Better pose adaptation
   - More flexible clothing combinations

### Mini Program Features

- Home page for product browsing
- AI virtual try-on page
- Shopping cart
- Order tracking
- User center

## Usage

### Web Interface

1. Start the Gradio service
2. Select a person image and a garment image
3. Choose the model mode
4. Adjust generation settings
5. Run inference and review the result

### Mini Program

1. Open the virtual try-on page
2. Upload a person image
3. Select or upload a clothing image
4. Wait for AI processing
5. Review the generated try-on result

## Model Setup

You need to download the required pretrained checkpoints manually, including:

- OOTDiffusion weights in `checkpoint/ootd/`
- CLIP model in `checkpoint/clip-vit-large-patch14/`
- VAE model files
- OpenPose checkpoints
- Human Parsing checkpoints

### Recommended Settings

- Input size: `768x1024`
- Output size: `768x1024`
- Inference steps: `20-40`
- Guidance scale: `1.0-5.0`

## Performance Notes

- GPU acceleration is strongly recommended
- You can balance speed and quality by adjusting inference steps
- Batch generation can be added for larger workloads

## Common Issues

1. Out-of-memory errors
   - Reduce the number of samples
   - Lower the input resolution

2. Unsatisfactory output quality
   - Increase the number of inference steps
   - Adjust the guidance scale
   - Use clearer input images

3. Mini program cannot reach the back end
   - Check whether the service is running
   - Verify the configured API endpoint

## License

This project is intended for learning and research purposes only.

## References

- [OOTDiffusion paper](https://arxiv.org/abs/2308.07269)
- [Gradio documentation](https://gradio.app/docs/)
- [WeChat Mini Program documentation](https://developers.weixin.qq.com/miniprogram/dev/framework/)

## Contact

Issues and pull requests are welcome for improvements and bug fixes.
