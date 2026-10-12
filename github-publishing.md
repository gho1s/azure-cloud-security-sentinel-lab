# Publishing this repository on GitHub

1. Review **every screenshot** for personal names, email addresses, tenant/subscription IDs, resource IDs, access tokens, IP addresses that should remain private, and organization details. Screenshots were cropped in the original package, but cropping alone is not a complete privacy review.
2. Create a new **public** GitHub repository named `azure-cloud-security-sentinel-lab` (or private first while reviewing).
3. Extract the ZIP locally.
4. Upload the **contents of the extracted `azure-cloud-security-sentinel-lab` folder**, not the ZIP file itself. Keep `README.md` in the repository root and preserve subfolders.
5. Confirm the README renders its architecture diagram and screenshots; open a few image links and the `.kql` files.
6. Add a repository description, relevant topics (`azure`, `microsoft-sentinel`, `cloud-security`, `kql`, `azure-policy`, `siem`), and a GitHub link to your resume or LinkedIn.
7. For future changes, make descriptive commits such as `docs: add incident verification evidence`.

**Do not upload** Azure credentials, `.env` files, secrets, tokens, private keys, or exported full resource configurations containing sensitive data.
