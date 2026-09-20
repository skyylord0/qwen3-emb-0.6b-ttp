> [!IMPORTANT]
> 🤗 **This project is hosted on Hugging Face.**  
> → [View the complete repository on Hugging Face](https://huggingface.co/skyylord/qwen3-emb-0.6b-ttp)

# SentenceTransformer

This is a [sentence-transformers](https://www.SBERT.net) model trained. It maps sentences & paragraphs to a 1024-dimensional dense vector space and can be used for retrieval.

## Motivation
 
This model is my contribution to a research project conducted at [LORIA](https://www.loria.fr/) (supervised by Jean-Yves Marion). My part of the project focuses on automatically mapping offensive security instructions and snippets — extracted from CTI reports, CISA advisories, and CTF writeups — to their corresponding [MITRE ATT&CK](https://attack.mitre.org/) TTPs (Tactics, Techniques, and Procedures).

Beyond raw accuracy, a deliberate design goal was to keep the model small enough to run comfortably on local, resource-constrained setups (a single consumer GPU, or even CPU) rather than depending on a heavier general-purpose embedding API. This makes it practical to use in offline, air-gapped, or otherwise constrained security research and red-team environments where sending data to an external service isn't an option.

## Model Details

### Model Description
- **Model Type:** Sentence Transformer
- **Base model:** [Qwen3-Embedding-0.6B](https://huggingface.co/Qwen/Qwen3-Embedding-0.6B)
- **Maximum Sequence Length:** 32768 tokens
- **Output Dimensionality:** 1024 dimensions
- **Similarity Function:** Cosine Similarity
- **Supported Modality:** Text

- The training and evaluation data used for this model are available at [skyylord/mitre-attack-ttp-labeled-instructions](https://huggingface.co/datasets/skyylord/mitre-attack-ttp-labeled-instructions).

### Model Sources

- **Documentation:** [Sentence Transformers Documentation](https://sbert.net)
- **Repository:** [Sentence Transformers on GitHub](https://github.com/huggingface/sentence-transformers)
- **Hugging Face:** [Sentence Transformers on Hugging Face](https://huggingface.co/models?library=sentence-transformers)

### Full Model Architecture

```
SentenceTransformer(
  (0): Transformer({'transformer_task': 'feature-extraction', 'modality_config': {'text': {'method': 'forward', 'method_output_name': 'last_hidden_state'}}, 'module_output_name': 'token_embeddings', 'architecture': 'Qwen3Model'})
  (1): Pooling({'embedding_dimension': 1024, 'pooling_mode': 'lasttoken', 'include_prompt': True})
  (2): Normalize({})
)
```

## Usage

### Direct Usage (Sentence Transformers)

First install the Sentence Transformers library:

```bash
pip install -U sentence-transformers
```
Then you can load this model and run inference.
```python
from sentence_transformers import SentenceTransformer

# Download from the 🤗 Hub
model = SentenceTransformer("skyylord/qwen3-emb-0.6b-ttp")
# Run inference
queries = [
    'Mustang Panda initial payloads downloaded a Windows Installer MSI file that in turn dropped follow-on files leading to installation of PlugX during RedDelta Modified PlugX Infection Chain Operations.',
]
documents = [
    'Msiexec: Adversaries may abuse msiexec.exe to proxy execution of malicious payloads. Msiexec.exe is the command-line utility for the Windows Installer and is thus commonly associated with executing installation packages (.msi). The Msiexec.exe binary may also be digitally signed by Microsoft. Adversaries may abuse msiexec.exe to launch local or network accessible MSI files. Msiexec.exe can also execute DLLs. Since it may be signed and native on Windows systems, msiexec.exe can be used to bypass application control solutions that do not account for its potential abuse. Msiexec.exe execution may also be elevated to SYSTEM privileges if the AlwaysInstallElevated policy is enabled.',
    "Malicious File: An adversary may rely upon a user opening a malicious file in order to gain execution. Users may be subjected to social engineering to get them to open a file that will lead to code execution. This user action will typically be observed as follow-on behavior from [Spearphishing Attachment]. Adversaries may use several types of files that require a user to execute them, including .doc, .pdf, .xls, .rtf, .scr, .exe, .lnk, .pif, .cpl, .reg, and .iso. Adversaries may employ various forms of [Masquerading] and [Obfuscated Files or Information] to increase the likelihood that a user will open and successfully execute a malicious file. These methods may include using a familiar naming convention and/or password protecting the file and supplying instructions to a user on how to open it. While [Malicious File] frequently occurs shortly after Initial Access it may occur at other phases of an intrusion, such as when an adversary places a file in a shared directory or on a user's desktop hoping that a user will click on it. This activity may also be seen shortly after [Internal Spearphishing].",
    'Executable Installer File Permissions Weakness: Adversaries may execute their own malicious payloads by hijacking the binaries used by an installer. These processes may automatically execute specific binaries as part of their functionality or to perform other actions. If the permissions on the file system directory containing a target binary, or permissions on the binary itself, are improperly set, then the target binary may be overwritten with another binary using user-level permissions and executed by the original process. If the original process and thread are running under a higher permissions level, then the replaced binary will also execute under higher-level permissions, which could include SYSTEM. Another variation of this technique can be performed by taking advantage of a weakness that is common in executable, self-extracting installers. During the installation process, it is common for installers to use a subdirectory within the %TEMP% directory to unpack binaries such as DLLs, EXEs, or other payloads. When installers create subdirectories and files they often do not set appropriate permissions to restrict write access, which allows for execution of untrusted code placed in the subdirectories or overwriting of binaries used in the installation process. This behavior is related to and may take advantage of [DLL] search order hijacking. Adversaries may use this technique to replace legitimate binaries with malicious ones as a means of executing code at a higher permissions level. Some installers may also require elevated privileges that will result in privilege escalation when executing adversary controlled code. This behavior is related to [Bypass User Account Control]. Several examples of this weakness in existing common installers have been reported to software vendors. If the executing process is set to run at a specific time or during a certain event (e.g., system bootup) then this technique can also be used for persistence.',
]
query_embeddings = model.encode_query(queries)
document_embeddings = model.encode_document(documents)
print(query_embeddings.shape, document_embeddings.shape)
# [1, 1024] [3, 1024]

# Get the similarity scores for the embeddings
similarities = model.similarity(query_embeddings, document_embeddings)
print(similarities)
# tensor([[0.5365, 0.2941, 0.2619]])
```

## Evaluation
 
### Metrics
 
Evaluation is performed over a set of adversarial-behavior sentences, each paired with its ground-truth TTP, at both the technique level (parent ATT&CK technique, e.g. `T1059`) and the sub-technique level (e.g. `T1059.001`).
 
- **MRR** (Mean Reciprocal Rank): the average of `1 / rank_of_correct_answer` across all queries. Rewards not just finding the right TTP, but ranking it as close to the top as possible. Ranges 0–1, higher is better.
- **H@1** (Hit@1): the % of queries for which the correct TTP is the very top result.
- **H@3** (Hit@3): the % of queries for which the correct TTP appears somewhere in the top 3 results.
- **K@75% / K@90% / K@95%**: the minimum number of top-K candidates that need to be returned for the correct TTP to be included for 75% / 90% / 95% of queries, respectively. Lower is better — this is a practical measure of "how many candidates do I need to hand to a downstream reranker or LLM to be confident the right answer is in there."

### Results
 
**Technique level – Test (4203 instructions)**
 
| Model | MRR | H@1 | H@3 | K@75% | K@90% | K@95% |
|---|---|---|---|---|---|---|
| Qwen3-Embedding-0.6B (base) | 0.56 | 43% | 63% | 7 | 31 | 71 |
| Qwen3-Embedding-4B (base) | 0.65 | 53% | 72% | 4 | 17 | 40 |
| **Qwen3-Embedding-0.6B (this model, fine-tuned)** | **0.85** | **79%** | **89%** | **1** | **4** | **12** 

**Sub-technique level – Test (4203 instructions)**

| Model | MRR | H@1 | H@3 | K@75% | K@90% | K@95% |
|---|---|---|---|---|---|---|
| Qwen3-Embedding-0.6B (base) | 0.46 | 34% | 53% | 14 | 61 | 132 |
| Qwen3-Embedding-4B (base) | 0.56 | 43% | 63% | 7 | 28 | 74 |
| **Qwen3-Embedding-0.6B (this model, fine-tuned)** | **0.80** | **72%** | **87%** | **2** | **5** | **16** |
 
**Technique level – held-out CISA advisories (1535 instructions)**
 
| Model | MRR | H@1 | H@3 | K@75% | K@90% | K@95% |
|---|---|---|---|---|---|---|
| Qwen3-Embedding-0.6B (base) | 0.60 | 49% | 65% | 7 | 27 | 61 |
| Qwen3-Embedding-4B (base) | 0.69 | 58% | 76% | 3 | 14 | 34 |
| **Qwen3-Embedding-0.6B (this model, fine-tuned)** | **0.76** | **68%** | **81%** | **2** | **10** | **28** |

**Sub-technique level – held-out CISA advisories (1535 instructions)**

| Model | MRR | H@1 | H@3 | K@75% | K@90% | K@95% |
|---|---|---|---|---|---|---|
| Qwen3-Embedding-0.6B (base) | 0.51 | 38% | 57% | 10 | 49 | 102 |
| Qwen3-Embedding-4B (base) | 0.61 | 49% | 69% | 5 | 22 | 61 |
| **Qwen3-Embedding-0.6B (this model, fine-tuned)** | **0.70** | **59%** | **78%** | **3** | **16** | **48** |

**Sub-technique level – exact match, CTF set (102 instructions)**

| Model | MRR | H@1 | H@3 | K@75% | K@90% | K@95% |
|---|---|---|---|---|---|---|
| Qwen3-Embedding-0.6B (base) | 0.46 | 29% | 55% | 9 | 23 | 34 |
| Qwen3-Embedding-4B (base) | 0.58 | 43% | 64% | 5 | 15 | 22 |
| **Qwen3-Embedding-0.6B (this model, fine-tuned)** | **0.69** | **54%** | **82%** | **3** | **7** | **36** |

**Technique level, CTF set (102 instructions)**

| Model | MRR | H@1 | H@3 | K@75% | K@90% | K@95% |
|---|---|---|---|---|---|---|
| Qwen3-Embedding-0.6B (base) | 0.54 | 41% | 62% | 7 | 21 | 32 |
| Qwen3-Embedding-4B (base) | 0.64 | 51% | 70% | 4 | 8 | 19 |
| **Qwen3-Embedding-0.6B (this model, fine-tuned)** | **0.78** | **68%** | **87%** | **2** | **5** | **13** |
 
The held-out splits — CISA advisories and the CTF set — were never seen during training or hyperparameter selection, and are used to check that performance generalizes beyond the training data distribution.
 
Before fine-tuning, several candidate base embedding models were benchmarked zero-shot on this task, including `bge-m3`, `all-mpnet-base-v2`, `jina-embeddings-v3`, and `SecBERT`. Qwen3-Embedding consistently outperformed all of them, which is why it was selected as the base model for fine-tuning here. (A full comparison table for these base models may be added in a future update.)


## Training Details

### Training Dataset

* Size: 19,624 training samples
* Columns: <code>anchor</code>, <code>positive</code>, <code>negative_0</code>, <code>negative_1</code>, and <code>negative_2</code>
* Approximate statistics based on the first 1000 samples:
  |         | anchor                                                                             | positive                                                                             | negative_0                                                                           | negative_1                                                                           | negative_2                                                                           |
  |:--------|:-----------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|
  | type    | string                                                                             | string                                                                               | string                                                                               | string                                                                               | string                                                                               |
  | details | <ul><li>min: 8 tokens</li><li>mean: 27.64 tokens</li><li>max: 155 tokens</li></ul> | <ul><li>min: 43 tokens</li><li>mean: 232.99 tokens</li><li>max: 784 tokens</li></ul> | <ul><li>min: 48 tokens</li><li>mean: 228.69 tokens</li><li>max: 800 tokens</li></ul> | <ul><li>min: 37 tokens</li><li>mean: 230.89 tokens</li><li>max: 800 tokens</li></ul> | <ul><li>min: 37 tokens</li><li>mean: 218.65 tokens</li><li>max: 800 tokens</li></ul> |
* Samples:
  | anchor                                                                                                     | positive                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | negative_0                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | negative_1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | negative_2                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
  |:-----------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
  | <code>APT41 DUST collected data from victim Oracle databases using SQLULDR2.</code>                        | <code>Databases: Adversaries may leverage databases to mine valuable information. These databases may be hosted on-premises or in the cloud (both in platform-as-a-service and software-as-a-service environments). Examples of databases from which information may be collected include MySQL, PostgreSQL, MongoDB, Amazon Relational Database Service, Azure SQL Database, Google Firebase, and Snowflake. Databases may include a variety of information of interest to adversaries, such as usernames, hashed passwords, personally identifiable information, and financial data. Data collected from databases may be used for [Lateral Movement], [Command and Control], or [Exfiltration]. Data exfiltrated from databases may also be used to extort victims or may be sold for profit.</code> | <code>Email Collection: Adversaries may target user email to collect sensitive information. Emails may contain sensitive data, including trade secrets or personal information, that can prove valuable to adversaries. Emails may also contain details of ongoing incident response operations, which may allow adversaries to adjust their techniques in order to maintain persistence or evade defenses. Adversaries can collect or forward email from mail servers or clients.</code>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | <code>Data from Cloud Storage: Adversaries may access data from cloud storage. Many IaaS providers offer solutions for online data object storage such as Amazon S3, Azure Storage, and Google Cloud Storage. Similarly, SaaS enterprise platforms such as Office 365 and Google Workspace provide cloud-based document storage to users through services such as OneDrive and Google Drive, while SaaS application providers such as Slack, Confluence, Salesforce, and Dropbox may provide cloud storage solutions as a peripheral or primary use case of their platform. In some cases, as with IaaS-based cloud storage, there exists no overarching application (such as SQL or Elasticsearch) with which to interact with the stored objects: instead, data from these solutions is retrieved directly though the [Cloud API]. In SaaS applications, adversaries may be able to collect this data directly from APIs or backend cloud storage objects, rather than through their front-end application or interface (i.e., [Data from In...</code> | <code>Local Email Collection: Adversaries may target user email on local systems to collect sensitive information. Files containing email data can be acquired from a user's local system, such as Outlook storage or cache files. Outlook stores data locally in offline data files with an extension of .ost. Outlook 2010 and later supports .ost file sizes up to 50GB, while earlier versions of Outlook support up to 20GB. IMAP accounts in Outlook 2013 (and earlier) and POP accounts use Outlook Data Files (.pst) as opposed to .ost, whereas IMAP accounts in Outlook 2016 (and later) use .ost files. Both types of Outlook data files are typically stored in `C:\Users\<username>\Documents\Outlook Files` or `C:\Users\<username>\AppData\Local\Microsoft\Outlook`.</code>                                                                                                                                                                                                                                                               |
  | <code>APT41 DUST deleted various artifacts from victim systems following use.</code>                       | <code>File Deletion: Adversaries may delete files left behind by the actions of their intrusion activity. Malware, tools, or other non-native files dropped or created on a system by an adversary (ex: [Ingress Tool Transfer]) may leave traces to indicate to what was done within a network and how. Removal of these files can occur during an intrusion, or as part of a post-intrusion process to minimize the adversary's footprint. There are tools available from the host operating system to perform cleanup, but adversaries may use other tools as well. Examples of built-in [Command and Scripting Interpreter] functions include del on Windows, rm or unlink on Linux and macOS, and `rm` on ESXi.</code>                                                                             | <code>Data Destruction: Adversaries may destroy data and files on specific systems or in large numbers on a network to interrupt availability to systems, services, and network resources. Data destruction is likely to render stored data irrecoverable by forensic techniques through overwriting files or data on local and remote drives. Common operating system file deletion commands such as del and rm often only remove pointers to files without wiping the contents of the files themselves, making the files recoverable by proper forensic methodology. This behavior is distinct from [Disk Content Wipe] and [Disk Structure Wipe] because individual files are destroyed rather than sections of a storage disk or the disk's logical structure. Adversaries may attempt to overwrite files and directories with randomly generated data to make it irrecoverable. In some cases politically oriented image files have been used to overwrite data. To maximize impact on the target organization in operations where networ...</code> | <code>Disk Content Wipe: Adversaries may erase the contents of storage devices on specific systems or in large numbers in a network to interrupt availability to system and network resources. Adversaries may partially or completely overwrite the contents of a storage device rendering the data irrecoverable through the storage interface. Instead of wiping specific disk structures or files, adversaries with destructive intent may wipe arbitrary portions of disk content. To wipe disk content, adversaries may acquire direct access to the hard drive in order to overwrite arbitrarily sized portions of disk with random data. Adversaries have also been observed leveraging third-party drivers like [RawDisk] to directly access disk content. This behavior is distinct from [Data Destruction] because sections of the disk are erased instead of individual files. To maximize impact on the target organization in operations where network-wide availability interruption is the goal, malware used for wiping disk ...</code> | <code>Disk Structure Wipe: Adversaries may corrupt or wipe the disk data structures on a hard drive necessary to boot a system; targeting specific critical systems or in large numbers in a network to interrupt availability to system and network resources. Adversaries may attempt to render the system unable to boot by overwriting critical data located in structures such as the master boot record (MBR) or partition table. The data contained in disk structures may include the initial executable code for loading an operating system or the location of the file system partitions on disk. If this information is not present, the computer will not be able to load an operating system during the boot process, leaving the computer unavailable. [Disk Structure Wipe] may be performed in isolation, or along with [Disk Content Wipe] if all sectors of a disk are wiped. On a network devices, adversaries may reformat the file system using [Network Device CLI] commands such as `format`. To maximize impact on th...</code> |
  | <code>APT41 DUST disguised DUSTPAN as a legitimate Windows binary such as `w3wp.exe` or `conn.exe`.</code> | <code>Masquerade Task or Service: Adversaries may attempt to manipulate the name of a task or service to make it appear legitimate or benign. Tasks/services executed by the Task Scheduler or systemd will typically be given a name and/or description. Windows services will have a service name as well as a display name. Many benign tasks and services exist that have commonly associated names. Adversaries may give tasks or services names that are similar or identical to those of legitimate ones. Tasks or services contain other fields, such as a description, that adversaries may attempt to make appear legitimate.</code>                                                                                                                                                          | <code>System Binary Proxy Execution: Adversaries may bypass process and/or signature-based defenses by proxying execution of malicious content with signed, or otherwise trusted, binaries. Binaries used in this technique are often Microsoft-signed files, indicating that they have been either downloaded from Microsoft or are already native in the operating system. Binaries signed with trusted digital certificates can typically execute on Windows systems protected by digital signature validation. Several Microsoft signed binaries that are default on Windows installations can be used to proxy execution of other files or commands. Similarly, on Linux systems adversaries may abuse trusted binaries such as split to proxy execution of malicious commands.</code>                                                                                                                                                                                                                                                              | <code>Path Interception by Unquoted Path: Adversaries may execute their own malicious payloads by hijacking vulnerable file path references. Adversaries can take advantage of paths that lack surrounding quotations by placing an executable in a higher level directory within the path, so that Windows will choose the adversary's executable to launch. Service paths and shortcut paths may also be vulnerable to path interception if the path has one or more spaces and is not surrounded by quotation marks (e.g., C:\unsafe path with space\program.exe vs. "C:\safe path with space\program.exe"). (stored in Windows Registry keys) An adversary can place an executable in a higher level directory of the path, and Windows will resolve that executable instead of the intended executable. For example, if the path in a shortcut is C:\program files\myapp.exe, an adversary may create a program at C:\program.exe that will be run instead of the intended program. This technique can be used for persistence if executa...</code> | <code>Hijack Execution Flow: Adversaries may execute their own malicious payloads by hijacking the way operating systems run programs. Hijacking execution flow can be for the purposes of persistence, since this hijacked execution may reoccur over time. Adversaries may also use these mechanisms to elevate privileges or evade defenses, such as application control or other restrictions on execution. There are many ways an adversary may hijack the flow of execution, including by manipulating how the operating system locates programs to be executed. How the operating system locates libraries to be used by a program can also be intercepted. Locations where the operating system looks for programs/resources, such as file directories and in the case of Windows the Registry, could also be poisoned to include malicious payloads.</code>                                                                                                                                                                                     |
* Loss: [<code>CachedMultipleNegativesRankingLoss</code>](https://sbert.net/docs/package_reference/sentence_transformer/losses.html#cachedmultiplenegativesrankingloss) with these parameters:
  ```json
  {
      "scale": 20.0,
      "similarity_fct": "cos_sim",
      "mini_batch_size": 4,
      "gather_across_devices": false,
      "directions": [
          "query_to_doc"
      ],
      "partition_mode": "joint",
      "hardness_mode": null,
      "hardness_strength": 0.0
  }
  ```

### Training Hyperparameters
#### Non-Default Hyperparameters

- `per_device_train_batch_size`: 16
- `learning_rate`: 2e-05
- `num_train_epochs`: 1
- `fp16`: True
- `batch_sampler`: no_duplicates

#### All Hyperparameters
<details><summary>Click to expand</summary>

- `do_predict`: False
- `eval_strategy`: no
- `prediction_loss_only`: True
- `per_device_train_batch_size`: 16
- `per_device_eval_batch_size`: 8
- `gradient_accumulation_steps`: 1
- `eval_accumulation_steps`: None
- `torch_empty_cache_steps`: None
- `learning_rate`: 2e-05
- `weight_decay`: 0.0
- `adam_beta1`: 0.9
- `adam_beta2`: 0.999
- `adam_epsilon`: 1e-08
- `max_grad_norm`: 1.0
- `num_train_epochs`: 1
- `max_steps`: -1
- `lr_scheduler_type`: linear
- `lr_scheduler_kwargs`: None
- `warmup_ratio`: None
- `warmup_steps`: 0
- `log_level`: passive
- `log_level_replica`: warning
- `log_on_each_node`: True
- `logging_nan_inf_filter`: True
- `enable_jit_checkpoint`: False
- `save_on_each_node`: False
- `save_only_model`: False
- `restore_callback_states_from_checkpoint`: False
- `use_cpu`: False
- `seed`: 42
- `data_seed`: None
- `bf16`: False
- `fp16`: True
- `bf16_full_eval`: False
- `fp16_full_eval`: False
- `tf32`: None
- `local_rank`: -1
- `ddp_backend`: None
- `debug`: []
- `dataloader_drop_last`: False
- `dataloader_num_workers`: 0
- `dataloader_prefetch_factor`: None
- `disable_tqdm`: False
- `remove_unused_columns`: True
- `label_names`: None
- `load_best_model_at_end`: False
- `ignore_data_skip`: False
- `fsdp`: []
- `fsdp_config`: {'min_num_params': 0, 'xla': False, 'xla_fsdp_v2': False, 'xla_fsdp_grad_ckpt': False}
- `accelerator_config`: {'split_batches': False, 'dispatch_batches': None, 'even_batches': True, 'use_seedable_sampler': True, 'non_blocking': False, 'gradient_accumulation_kwargs': None}
- `parallelism_config`: None
- `deepspeed`: None
- `label_smoothing_factor`: 0.0
- `optim`: adamw_torch_fused
- `optim_args`: None
- `group_by_length`: False
- `length_column_name`: length
- `project`: huggingface
- `trackio_space_id`: trackio
- `ddp_find_unused_parameters`: None
- `ddp_bucket_cap_mb`: None
- `ddp_broadcast_buffers`: False
- `dataloader_pin_memory`: True
- `dataloader_persistent_workers`: False
- `skip_memory_metrics`: True
- `push_to_hub`: False
- `resume_from_checkpoint`: None
- `hub_model_id`: None
- `hub_strategy`: every_save
- `hub_private_repo`: None
- `hub_always_push`: False
- `hub_revision`: None
- `gradient_checkpointing`: False
- `gradient_checkpointing_kwargs`: None
- `include_for_metrics`: []
- `eval_do_concat_batches`: True
- `auto_find_batch_size`: False
- `full_determinism`: False
- `ddp_timeout`: 1800
- `torch_compile`: False
- `torch_compile_backend`: None
- `torch_compile_mode`: None
- `include_num_input_tokens_seen`: no
- `neftune_noise_alpha`: None
- `optim_target_modules`: None
- `batch_eval_metrics`: False
- `eval_on_start`: False
- `use_liger_kernel`: False
- `liger_kernel_config`: None
- `eval_use_gather_object`: False
- `average_tokens_across_devices`: True
- `use_cache`: False
- `prompts`: None
- `batch_sampler`: no_duplicates
- `multi_dataset_batch_sampler`: proportional
- `router_mapping`: {}
- `learning_rate_mapping`: {}

</details>

### Training Logs
| Epoch  | Step | Training Loss |
|:------:|:----:|:-------------:|
| 0.0994 | 122  | 0.2827        |
| 0.1989 | 244  | 0.2189        |
| 0.2983 | 366  | 0.2301        |
| 0.3977 | 488  | 0.2408        |
| 0.4971 | 610  | 0.2930        |
| 0.5966 | 732  | 0.3242        |
| 0.6960 | 854  | 0.3632        |
| 0.7954 | 976  | 0.4154        |
| 0.8949 | 1098 | 0.4609        |
| 0.9943 | 1220 | 0.5574        |


### Training Time
- **Training**: 7.0 hours

### Framework Versions
- Python: 3.12.13
- Sentence Transformers: 5.4.0
- Transformers: 5.0.0
- PyTorch: 2.10.0+cu128
- Accelerate: 1.13.0
- Datasets: 4.8.5
- Tokenizers: 0.22.2

## Citation

### BibTeX

If you use this model, please cite it as follows:
```bibtex
@misc{akhlafa2026mitrettp,
  title        = {Qwen3-Embedding Fine-tuned for MITRE ATT\&CK TTP Mapping},
  author       = {Akhlafa Amine},
  year         = {2026},
  howpublished = {\url{https://huggingface.co/skyylord/qwen3-emb-0.6b-ttp}},
}
```

#### Sentence Transformers
```bibtex
@inproceedings{reimers-2019-sentence-bert,
    title = "Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks",
    author = "Reimers, Nils and Gurevych, Iryna",
    booktitle = "Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing",
    month = "11",
    year = "2019",
    publisher = "Association for Computational Linguistics",
    url = "https://arxiv.org/abs/1908.10084",
}
```

#### CachedMultipleNegativesRankingLoss
```bibtex
@misc{gao2021scaling,
    title={Scaling Deep Contrastive Learning Batch Size under Memory Limited Setup},
    author={Luyu Gao and Yunyi Zhang and Jiawei Han and Jamie Callan},
    year={2021},
    eprint={2101.06983},
    archivePrefix={arXiv},
    primaryClass={cs.LG}
}
```

<!--
## Glossary

*Clearly define terms in order to be accessible across audiences.*
-->

<!--
## Model Card Authors

*Lists the people who create the model card, providing recognition and accountability for the detailed work that goes into its construction.*
-->

<!--
## Model Card Contact

*Provides a way for people who have updates to the Model Card, suggestions, or questions, to contact the Model Card authors.*
-->
