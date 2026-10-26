# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
52
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->251.9<!--/answer--> | <!--answer:vm_warm_median_ms-->209.9<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->148.1<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->636.0<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->182.3<!--/answer--> | <!--answer:function_warm_median_ms-->667.7<!--/answer--> |
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
[OK  ] vm        cold=  251.9ms  warm_median=  209.9ms
             #############
[OK  ] cloudrun  cold=  148.1ms  warm_median=  636.0ms
             ######################################
[OK  ] function  cold=  182.3ms  warm_median=  667.7ms
             ########################################
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The IaaS VM creates nothing automatically; load is absorbed by a single static instance constrained strictly by the hardware limits of the `--machine-type=e2-micro` size chosen. Cloud Run automatically scales container instances up and down based on incoming request volume, bounded by the `--max-instances=3` flag set during deployment. Cloud Functions similarly autoscales serverless instances automatically managed by Google Cloud infrastructure, governed by platform concurrency limits.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
For the IaaS VM, I had to manually build a custom VPC, configure subnets, open firewall rules for SSH and port 8080, provision the VM instance, SSH in, install and authenticate Docker, and run the container detached. For Cloud Run, deployment was abstracted via Cloud Build and managed PaaS container deployment commands. For Cloud Functions, no containerization was necessary locally; deploying raw Python source code directly handled the build and serverless runtime setup automatically.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
NAME           REGION       URL                                                 STATUS
echo-bot       us-central1  https://echo-bot-1056378553243.us-central1.run.app  READY
echo-function  us-central1  https://echo-function-byuqostqua-uc.a.run.app      READY
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output of `gcloud run services list` reveals that Cloud Functions gen 2 is built directly on top of Cloud Run under the hood. When you deploy a 2nd-generation Cloud Function from source, Google Cloud automatically packages it and provisions a corresponding underlying Cloud Run service. This explains why both services exhibit nearly identical architectural behavior, container execution models, and comparable cold/warm startup latencies. The primary difference is the developer experience: Cloud Run expects a pre-built container image, whereas Cloud Functions accepts raw source code and manages the build pack and runtime environment automatically.
<!--/answer-->
