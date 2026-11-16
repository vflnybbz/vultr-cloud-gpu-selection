# Vultr Cloud GPU: Which of the 11 GPU Families Fits Your Workload, What A100 and L40S Cost Per Hour, and the Provisioning Limits to Check First

Vultr Cloud GPU lists 11 GPU families, and only two of them publish a starting hourly price on Vultr's own product pages: NVIDIA A100 PCIe at **$1.290/hour** and NVIDIA L40S at **$1.671/hour**. The other nine, including HGX B200, H100/H200, GH200 and the three AMD Instinct cards, don't show a public starting rate on the pages I could verify, so the real number has to come off the plan list inside the deploy flow.

What follows is the full lineup in one table, a three-step rule for picking a card, the exact provisioning path, and three limits that appear only in the documentation.

> Most readers end up choosing between two plans. If your job needs 80GB of GPU memory and a plain CUDA box for training or inference, that's A100 PCIe at **$1.290/hour**. If you need FP8 throughput, newer Ada Lovelace silicon and graphics capability on the same card, that's L40S at **$1.671/hour** with 48GB. Anything above those two is quoted per plan at deploy time.
>
> 👉 [Check the A100 PCIe starting price and available plans](https://www.vultr.com/products/cloud-gpu/nvidia-a100/?ref=9738262-9J)

## The full Vultr Cloud GPU lineup

Six columns, one row per family. Where Vultr's pages don't state a spec, the cell shows "—" rather than a guess.

| GPU family | Architecture | GPU memory | Deployment form | Starting price (USD/hr) | Details |
| --- | --- | --- | --- | --- | --- |
| NVIDIA A16 | Ampere | 64 GB | vGPU | See current pricing | [A16 plans](https://www.vultr.com/products/cloud-gpu/nvidia-a16/?ref=9738262-9J) |
| NVIDIA A40 | Ampere | 48 GB | vGPU | See current pricing | [A40 plans](https://www.vultr.com/products/cloud-gpu/nvidia-a40/?ref=9738262-9J) |
| NVIDIA A100 PCIe | Ampere | 80 GB HBM2e | vGPU | **$1.290** | [A100 PCIe plans](https://www.vultr.com/products/cloud-gpu/nvidia-a100/?ref=9738262-9J) |
| NVIDIA HGX A100 | Ampere | 80 GB HBM2e | SXM (HGX) | See current pricing | [HGX A100 plans](https://www.vultr.com/products/cloud-gpu/nvidia-a100/?ref=9738262-9J) |
| NVIDIA L40S | Ada Lovelace | 48 GB | Passthrough | **$1.671** | [L40S plans](https://www.vultr.com/products/cloud-gpu/nvidia-l40s/?ref=9738262-9J) |
| NVIDIA HGX H100 | Hopper | — | SXM (HGX) | See current pricing | [H100/H200 plans](https://www.vultr.com/products/cloud-gpu/nvidia-h100-h200/?ref=9738262-9J) |
| NVIDIA GH200 | Grace Hopper superchip | — | Superchip module | See current pricing | [GH200 plans](https://www.vultr.com/products/cloud-gpu/nvidia-gh200/?ref=9738262-9J) |
| NVIDIA HGX B200 | Blackwell | Up to 192 GB HBM3e per GPU | HGX, 8 GPUs per system | See current pricing | [B200 details](https://www.vultr.com/products/cloud-gpu/nvidia-b200/?ref=9738262-9J) |
| AMD Instinct MI300X | — | — | — | See current pricing | [MI300X plans](https://www.vultr.com/products/cloud-gpu/amd-mi300x/?ref=9738262-9J) |
| AMD Instinct MI325X | — | — | — | See current pricing | [MI325X plan pricing](https://bit.ly/VuLtr) |
| AMD Instinct MI355X | CDNA 4 | 288 GB HBM3e | — | See current pricing | [MI355X plans](https://www.vultr.com/products/cloud-gpu/amd-mi355x/?ref=9738262-9J) |

"—" means the Vultr pages I reviewed don't state that value. Prices are US dollars per hour and are the *starting* rates shown on the product pages; the A100 page also renders the same figure as "$1.29 per hour." Plan-level configurations (vCPU, RAM, storage, bandwidth) vary by family and are only visible after you pick a family in the deploy flow.

### Cores and TF32 for the four cards Vultr specs out

Vultr publishes a comparison table for four of the 11 families. These are the numbers:

| GPU | CUDA cores | Tensor cores | TF32 TFLOPS (with sparsity) | Memory |
| --- | --- | --- | --- | --- |
| NVIDIA A16 | 5,120 | 160 | 72 | 64 GB |
| NVIDIA A40 | 10,752 | 336 | 149.6 | 48 GB |
| NVIDIA A100 | 6,912 | 432 | 312 | 80 GB |
| NVIDIA L40S | 18,716 | 568 | 366 | 48 GB |

The L40S page goes further into precision: FP32 91.6 TFLOPS, TF16 733, **FP8 1,466 TFLOPS (with sparsity)**, RT Core 212, max power 350W. One caution on that page: a marketing line elsewhere on it quotes much larger "Tensor performance" figures, which are FP8-with-sparsity numbers, not single-precision. Use the spec table.

## How to pick a card: memory, then precision, then form factor

Work down these three questions in order. Most bad GPU purchases come from answering them in the wrong order.

**1. Does the model fit in memory?** 80GB on A100, 48GB on L40S, up to 192GB HBM3e per GPU on B200, 288GB HBM3e on MI355X. If weights plus optimizer state plus activations exceed the card, no TFLOPS figure rescues the run. This single number decides more deployments than any benchmark.

**2. Does the precision you need exist on the card?** The A100 spec sheet lists FP64, TF32, BF16/FP16 and INT8, with no FP8 entry. L40S carries FP8 at 1,466 TFLOPS with sparsity and a Transformer Engine that switches between FP8 and FP16 layer by layer. B200 adds FP4, with Vultr quoting **144 petaFLOPS FP8 (with sparsity)** across an eight-GPU HGX system. If you're serving transformer inference at volume, FP8 changes the cost per token; if you're doing double-precision simulation, the A100's FP64 9.7 TFLOPS matters more.

**3. How is the GPU handed to your VM?** Vultr's docs define two modes, and the difference affects both performance and how many tenants can share a card:

- **vGPU** partitions one physical GPU across multiple VMs.
- **Passthrough** gives an entire GPU to a single VM, for maximum performance and compatibility.

That's why the lineup mixes modes: A16, A40 and the A100 entry are listed as vGPU, while L40S is listed as Passthrough. If you're reselling slices or running many small workloads, vGPU economics win. If you need the whole card's bandwidth and full driver compatibility, Passthrough is the safer bet.

> 👉 [Compare the L40S plan list and current hourly rate](https://www.vultr.com/products/cloud-gpu/nvidia-l40s/?ref=9738262-9J)

### A100 PCIe and HGX A100 are not the same purchase

"A100" appears twice in the lineup for a reason. The PCIe version connects through the PCIe interface and ships in a form compatible with standard servers; the SXM version sits in an HGX chassis with better power delivery and thermal management, which Vultr's docs credit with higher performance. Both carry 80GB HBM2e. The spec sheet separates them further: memory bandwidth is 1,935 GB/s on PCIe versus 2,039 GB/s on SXM, and max TDP is 300W versus 400W. (The FAQ on the same page says "up to 1.6 TB/s," which doesn't match either figure in the table. Trust the table, and note which form factor you're buying.)

Both support MIG, up to **7 instances at 10GB each**, which is how you serve several small models on one card without them fighting over memory. NVLink runs at 600 GB/s. Vultr's product page says A100 is available virtualized or as bare metal; the docs comparison table lists the A100 entry as vGPU. Treat "A100 PCIe" and "HGX A100" as the two things you're choosing between, not "A100" as one SKU.

## Which card for which job

- **Virtual desktops, video transcoding, light inference**: A16. It exists for low-latency VDI and inference, and it's the cheapest way to get many small GPU slices.
- **3D rendering, virtual production, simulation**: A40, built for advanced visualization work.
- **General AI training and inference that needs 80GB**: A100 PCIe. The 7-way MIG support also makes it the flexible choice when one box has to serve several teams.
- **Multimodal and LLM inference where FP8 and graphics both matter**: L40S. It also supports MIG and SR-IOV for slicing and isolation, meets NEBS Level 3, and has secure boot with root of trust. NVIDIA's own comparison claims up to 1.7x training and 1.5x inference over A100; that's a vendor figure, not a Vultr measurement.
- **Very large training runs that need FP4 and 192GB per GPU**: HGX B200. Read the specs as preliminary, because the page carries a "may be subject to change" disclaimer. Vultr also repeats NVIDIA's claims of up to 15x real-time inference and 3x training versus the Hopper generation, again vendor numbers.
- **AMD route**: MI300X for AI and HPC with high memory bandwidth, MI325X for memory-hungry training and inference, MI355X (CDNA 4, 288GB HBM3e, FP8/FP4) for the largest models.
- **Superchip form factor**: GH200, if your stack is built around the Grace Hopper combination rather than a standalone GPU.

Two quick answers that come up before ordering:

**Do I install CUDA drivers myself?** No. GPU-enabled OS images in the deploy flow ship with preinstalled drivers.

**Can one GPU serve several workloads?** On A100, MIG gives you up to seven 10GB instances. L40S supports MIG and SR-IOV slicing. On vGPU families, the card is already partitioned by design.

## Provisioning a Cloud GPU instance

The flow is short, but step order matters:

1. Go to **Compute → Instances**, then click **Create Instance +**.
2. Under **Plan**, choose **Cloud GPU**.
3. Pick a GPU family in the sidebar (for example NVIDIA A100 PCIe, NVIDIA HGX B200 or AMD MI300X).
4. Choose a plan by vCPU, memory, storage, bandwidth and price. Plans that boot from Block Storage show **Bootable Block Volume** in the Storage column instead of a fixed local disk size.
5. Choose a **Location**.
6. Click **Configure**, then pick an image: Operating System (Ubuntu, Debian, CentOS, Rocky Linux and more; GPU-enabled versions include preinstalled drivers), Marketplace App, iPXE, Backup, Snapshot or ISO.
7. Under **Server Settings**, optionally set an IP address, SSH keys, a startup script and a firewall group.
8. Fill in the hostname and label, set Additional Features (public IP or private behind a NAT gateway, VPC network, automatic backups, DDoS protection, limited user login, Cloud-Init user data).
9. Review the summary and click **Deploy**.

## Three limits on block-storage-boot GPU plans

These are documented, easy to miss, and they change how you script a deployment:

> - The boot volume and the instance **must be in the same location**.
> - **No Startup Script option** on these plans.
> - **No Automatic Backups** on these plans.

You also have to choose an attachment method: **Attach** uses the existing bootable volume, **Clone** clones it and leaves the source unattached. If your provisioning depends on cloud-init style startup scripts or scheduled backups, pick a plan with local storage instead of a bootable block volume.

> 👉 [See every Cloud GPU family and its plan-level hourly rate](https://bit.ly/VuLtr)

## Before you click Deploy

Three things to confirm on the order screen, because they aren't published where you'd expect: the per-plan hourly rate for whichever family you picked (only A100 PCIe and L40S have public starting prices), whether that family is in stock in your target location, and how your account credit or promo balance applies to GPU plans. Promotional credit eligibility is set by Vultr's billing terms at signup, and it's worth reading before you size a plan around it.

The selection rule, condensed:

- Model fits in 48GB and you want FP8 plus rendering on one card → **L40S, $1.671/hour**.
- Model needs 80GB, or you want to carve one card into seven slices for multiple teams → **A100 PCIe, $1.290/hour**.
- You need 192GB per GPU, FP4, or an eight-GPU NVLink domain → **HGX B200**, specs preliminary, price from the plan list.
- You need many small GPU slices for desktops or transcoding → **A16**.
- Rendering and simulation lead your workload → **A40**.

If the first two rows describe your job, start there: 👉 [Open the A100 PCIe product page and current hourly pricing](https://www.vultr.com/products/cloud-gpu/nvidia-a100/?ref=9738262-9J).
