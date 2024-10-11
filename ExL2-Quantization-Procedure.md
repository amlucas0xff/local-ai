**ExL2 Quantization Procedure (Using Oobabooga's Text Generation UI)**

**Introduction:**  
This guide is here to help Linux users learn how to quantize LLMs (large language models) with as little hassle as possible, using the Text Generation UI tool. The goal is to let you tweak your models to fit your available VRAM and make them run efficiently without getting bogged down in errors.

**What is Quantization?**  
Quantization is about converting a model from a higher precision (like FP16) to a lower precision (like INT8). Why do this? It makes the model smaller and more efficient, which is great for hardware with limited resources like GPUs with lower VRAM or edge devices. It also means faster models that use less memory—perfect for real-time applications.

**Why Should You Know How to Quantize Models?**  
Learning how to quantize models gives you the power to optimize them specifically for your hardware. It means you can make the most out of what you have without relying on someone else to release a pre-quantized version that might not fit your needs. This skill gives you independence, allowing you to customize models and work more efficiently.

**Why I Choose to Use ExL2 Instead of Other Formats**  
ExL2 format offers a unique blend of performance and flexibility that makes it particularly well-suited for quantizing large language models. ExL2 allows you to pick fractional bits per weight (bpw), which means you can fine-tune the quantization level based on your specific VRAM limitations and performance goals. This flexibility is ideal for those who want to get the most out of their GPU without sacrificing model accuracy.

> [!NOTE]
> Larger VRAM allows for more bits per weight, which means better model accuracy and performance. Understanding this relationship will help you decide how to effectively quantize your models for your specific hardware setup.

**Prerequisites:**  
- Oobabooga's Text Generation UI tool installed
- Basic familiarity with Oobabooga's Text Generation UI (installation, loading, and inference).

**Step-by-Step Workflow**

1. **Install Text Generation UI**  
   - Download the latest release of Oobabooga's Text Generation UI from the official repository.
   - Run the installation script suitable for your operating system.
   - If it's already installed, make sure to update to the latest version using the respective update script.

2. **Initial Setup and Model Testing**  
   - Start the Text Generation UI using the startup script for Linux (e.g., `start_linux.sh`).
   - Test if you can load ExLLaMAv2's exl2 models by downloading a pre-quantized model from HuggingFace. Load it with the "exllamav2_HF loader".

3. **Obtain an Unquantized Model**  
   - Visit Hugging Face and find an unquantized model (usually labeled as FP16/BF16 without any quantization format).
   - Download all the files to `text-generation-webui/models/`. A good starting model is "open_llama_3b" (approx. 6.5GB unquantized).

4. **Activate the Required Environment**  
   - Find the command-line script (`cmd_linux.sh`) in the Text Generation WebUI directory.
   - Run it to activate Oobabooga's Conda environment, which contains everything you need for quantization.

5. **Convert Model to SafeTensors Format**  
   - If the model is in `.bin` (Pickle) format, convert it to `.safetensors` using:
     ```
     python convert-to-safetensors.py input_path -o output_path
     ```
     - Example:
     ```
     python convert-to-safetensors.py models/open_llama_3b -o models/open_llama_3b_fp16
     ```
   - If your model is already in `.safetensors`, skip to step 7.

6. **Verify Model Conversion**  
   - Load the `.safetensors` model in TextGen UI's "Transformers Loader" to confirm the conversion was successful.
   - Unload the model before moving on.

7. **Download ExLLaMAv2 Release**  
   - Download the latest release of ExLLaMAv2 from the repository. Use a version that matches the TextGen UI dependencies (e.g., "Source Code.zip" for version `0.0.13post2`).
   - Extract it into the `text-generation-webui` directory, avoiding spaces in the path.
   - Create a folder named `working` inside this directory to store temporary files during quantization.

8. **Navigate to ExLLaMAv2 Directory**  
   - Change directory to the ExLLaMAv2 folder:
     ```
     cd exllamav2-0.0.13.post2
     ```

9. **Prepare for Quantization**  
   - ExLLaMAv2 uses a measurement-based quantization method, running some calibration data to measure quantization loss.
   - For more aggressive quantization (less than 4 bpw) or specific use cases, use a custom calibration dataset (in `.parquet` format) from HuggingFace, specifying the path using the `-c` argument.

10. **Run the Quantization**  
    - Keep an eye on RAM and VRAM usage during quantization with system tools (e.g., `htop`,`btop`).
    - Run the quantization command in the Conda terminal:
      ```
      python convert.py -i input_path -o working_path -cf output_path -hb head_size -b bpw
      ```
      - Example Command:
      ```
      python convert.py -i ../models/open_llama_3b_fp16 -o working -cf ../models/open_llama_3b_exl2 -b 6 -hb 8 -nr
      ```
      - Parameters:
        - **`-i`**: Input path to the `.safetensors` model.
        - **`-o`**: Directory for temporary files.
        - **`-cf`**: Output path for the quantized model.
        - **`-hb`**: Head bit size (`8` for `bpw >= 6`, otherwise `6`).
        - **`-b`**: Bits per weight for the majority of layers.
        - **`-nr`**: Optional flag to flush working files before starting.

11. **Completion and Usage**  
    - Once quantization is done, load the newly quantized model (`exl2`) in TextGen UI.
    - Enjoy a more efficient model that fits your VRAM.
