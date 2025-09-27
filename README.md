# deepfake32_on_KV260

Implementation of a **Deepfake32 hardware accelerator** on AMD Kria KV260, using Vitis HLS (for IP generation) and Vivado (for block design).
Achieved **~190× speedup** over GPU baseline inference.

---

## 1. Environment setup

* **Host Machine:** Ubuntu 20.04 (PC/laptop with Vivado & Vitis HLS installed)
* **Target Board:** AMD Kria KV260 (Ubuntu 22.04 image installed directly from Kria Starter Kit)
* **Runtime:** XRT (Xilinx Runtime) installed on KV260

---

## 2. HLS IP Creation (GUI workflow)

1. Open **Vitis HLS**.
2. Create new project → add your `deepfake32` C/C++ source.
3. Set top function as `deepfake32_top`.
4. Configure part: KV260 SOM device (ZU5EV).
5. Run **C Simulation** → verify testbench.
6. Run **C Synthesis** → review timing/resource utilization.
7. Export RTL → **IP Catalog format**.

   * Output: `deepfake32.zip`

---

## 3. Vivado Block Design

1. Launch **Vivado** (GUI).
2. Create new project → select KV260 board preset.
3. Add **HLS IP** (`deepfake32`) to the IP repository.
4. Create Block Design:

   * Zynq UltraScale+ MPSoC (Processing System)
   * AXI Interconnect
   * deepfake32 HLS IP
   * (Optional) AXI DMA for data transfer
5. Connect clocks, resets, AXI interfaces.
6. Validate design → Generate Output Products.
7. Run **Synthesis → Implementation → Generate Bitstream**.
8. Export Hardware with Bitstream (`deepfake32.xsa`).

---

## 4. Deployment on KV260 (Ubuntu)

1. Install **XRT** on KV260:

```bash
sudo apt update
sudo apt install xrt
```

2. Copy files to board:

```bash
scp deepfake32.xclbin kv260:/home/ubuntu/
```

3. Load accelerator bitstream:

```bash
sudo xbutil program --bitstream deepfake32.xclbin
```

4. Verify with:

```bash
xbutil validate
```

---

## 5. Running inference

* Use a Python or C++ host program (`host/run_fpga.py`) that:

  * Allocates buffers
  * Sends input to FPGA
  * Triggers accelerator
  * Reads output
* Example (Python skeleton):

```python
import pynq
ol = pynq.Overlay("deepfake32.xclbin")
accel = ol.deepfake32_0

# allocate buffers, send input, run, read output
```

---

## 6. Performance Measurement

* **GPU baseline (PyTorch, CUDA):** measure average latency.
* **FPGA runtime (KV260):** measure accelerator latency (input+compute+output).
* Speedup = `GPU_time / FPGA_time`
* Observed: **~190× faster than GPU inference.**

---

## 7. Results

* FPGA achieved near-real-time inference.
* Power efficiency improved significantly vs GPU.
* Demonstrated viability of FPGA accelerators for AI workloads.

---

