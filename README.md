# Proposal - Post-Quantum Blockchain: Thử nghiệm chữ ký Dilithium / Falcon cho Smart Contract.

## 1. Asset‑Centric Context (AIM).
Trong bối cảnh Post-Quantum Blockchain - Thử nghiệm và đánh giá chữ ký Dilithium/ Falcon cho Smart Contract, các tài sản (assets) được phân loại theo nhóm mục tiêu bảo mật (Data /Key /Identity /State /Infrastructure) như sau: 
### 1.1 Danh mục tài sản.
#### A1. Dữ liệu cần bảo vệ:
- in-transit: Dữ liệu giao dịch (transaction data).
- at-rest: Dữ liệu smart contract và các dữ liệu on-chain khác.
- in-process: Quá trình verify on-chain.


#### A2) Bí mật và Khóa (Secrets and Keys).
- Private key PQC.
- Public key PQC.
- ECDSA keys (nhằm mục đích so sánh).
- Seed cho zk-proof (nếu dùng).
- KMS/ Vault root keys.

#### A3) Danh tính (Identities).
- User Identity: Externally Owned Accounts (EOAs)
- Contract Account Identity.
- Node/Validator Identity.
- Others: (Multi-Signature Wallets,  Verifiable Credentials)
#### A4) Trạng thái và Chính sách (State and Policy Assets).
- Blockchain State: Ownership records, System configs.
- Access Policy / Authorization Policy: Smart contract rules.
- Attestation Policy (off-chain).

#### A5) Hạ tầng in cậy (Trusted Infrastructure).
- PQC validator.
- Blockchain Node / Runtime.
- PQ Signature Library (PQClean / liboqs).
- CA / Attestation Root (nếu có).


### 1.2 Ngữ cảnh và ràng buộc ( Architecture and Constraints). 

#### Hệ thống
1. Môi trường Blockchain: Ethereum (EVM), CosmWasm (WASM).
2. Signer (Client-side): Off-chain, sinh khóa, ký và lưu trữ khóa.
3. Smart contract (On-chain, trusted): thực thi logic (Xác minh chữ ký, thực thi nghiệp vụ).
4. Off-chain Verifier / Relayer: Phụ trợ, giảm chi phí on-chain.

#### Trust Boundaries
- Off-chain vs On-chain (Boundary chính): Mọi dữ liệu từ off-chain lên on-chain đều phải được xác thực nghiêm ngặt.
- On-chain runtime: Giữa mã nguồn smart contract và môi trường thực thi (EVM/WASM).

#### Ràng buộc kỹ thuật và ràng buộc hiệu năng
- Gas limit: Chi phí tính toán PQC trên EVM rất cao.
- Transaction size: Kích thước khóa lớn dẫn đến kích thước giao dịch tăng theo.
- Latency: Các phương pháp off-chain khiến độ trễ tăng cao.
- Môi trường thực thi: EVM không được thiết kế để thực hiện tính toán số học phức tạp khiến PQC khó triển khai.

---

### **1.3. Phân tích Rủi ro & Mục tiêu Bảo vệ (Risk Analysis & SMART Protection Goals)**

#### **Rủi ro chính & Biện pháp giảm thiểu (Key Risks & Mitigations)**

| ID | Rủi ro chính (Risk) | Mô tả ngắn gọn | Biện pháp giảm thiểu tương ứng (Mitigation) |
| :--- | :--- | :--- | :--- |
| R1 | **Giả mạo hậu lượng tử(Post-Quantum Forgery)** | ECDSA bị phá bởi máy tính lượng tử dẫn đến giả mạo danh tính. | **Sử dụng Chữ ký Hậu-lượng-tử (PQC Signatures)** |
| R2 | **Lỗi logic trình xác thực (Verifier Logic Flaw)** | Lỗi triển khai logic trong hệ thống dẫn đến: <br/>- Chấp nhận chữ ký sai (false-accept) <br/> - từ chối  chữ ký đúng (false-reject). | **Test và thử nghiệm chéo trên các bộ thư viện chuẩn:** Kiểm tra chéo logic xác thực on-chain với known-answer tests. |
| R3 | **Tấn công Từ chối Dịch vụ (Gas-based Denial of Service)** | Chi phí gas quá cao, dẫn đến việc bị spam giao dịch gây gián đoạn xử lý cho các node. | **Tối ưu Chi phí tính toán:** Thử nghiệm và so sánh các mô hình thay thế như xác thực off-chain + chứng thực on-chain (attestation), zk-proofs,... để giảm thiểu tính toán on-chain. |
| R4 | **Chi phí Dữ liệu cao (Data Cost Overhead)** | Kích thước khóa lớn gây vấn đề đội chi phí cho người dùng.| **Lựa chọn Tham số phù hợp:** Kiểm thử, đánh giá các tham số triển khai của Dilithium/Falcon. Chọn tham số phù hợp. |
| R5 | **Tấn công Kênh bên (Side-Channel Attack)** | Quá trình tạo chữ ký PQC (đặc biệt là Falcon với thuật toán sampling phức tạp) ở phía client có thể bị tấn công kênh phụ, làm rò rỉ khóa riêng. | **Triển khai đúng, sử dụng thư viện đảm bảo an toàn:** PQClean/liboqs, constant-time, chú ý trong triển khai Falcon... |

---

#### **Mục tiêu bảo vệ định lượng (SMART Protection Goals)**


| ID | Mục tiêu Bảo vệ (Protection Goal) | Metric (Chỉ số đo lường) | Ngưỡng chấp nhận (Threshold) |
| :--- | :--- | :--- | :--- |
| G1 | **Đảm bảo tính chính xác mật mã.** Hệ thống phải xác thực đúng chữ ký PQC, tránh false-accept và false-reject. | False-Accept Rate với các chữ ký không hợp lệ. | **0%** |
| G2 | **Đảm bảo tính khả thi về chi phí thực thi on-chain.** | Chi phí gas trung bình mỗi lần verify trên EVM. | **$\leq$ 15 triệu gas**.[^1] |
| G3 | **Giảm chi phí Off‑chain verification**  | Tỷ lệ giảm chi phí gas so với phương pháp xác thực on-chain trực tiếp. | Chưa xác định |
| G4 | **Kiểm soát chi phí dữ liệu giao dịch.** | Kích thước giao dịch | Chưa xác định |
| G5 | **Đánh giá các vấn đề bảo mật.**| Kiểm tra, đánh giá các rủi ro liên quan tới side-channel/constant-time attack, độ phức tạp code và rủi ro lỗi logic. | Không có tiêu chuẩn cụ thể. |

## 2) System Architecture (đề suất).
### 2.1 Sơ đồ kiến trúc: 

|![image](https://hackmd.io/_uploads/HJCxA1dalx.png)<br><center>Overall Blockchain process</center>|
---

|![image](https://hackmd.io/_uploads/S11hA1Ople.png) <center>**Scenario 1: Naive On-chain verification**</center>|
---

|![image](https://hackmd.io/_uploads/r1A3lxOpgg.png)<center>**Scenario 2: Off-chain verification/On-chain attestation**</center>|
--

|![image](https://hackmd.io/_uploads/HJQh-e_axx.png)<center>**Scenario 3: Prover - zk-proof**</center>|
--
### 2.2 Thành phần lõi và Quyết định kỹ thuật.
1. Signer (off-chain): Sử dụng PQClean/liboqs. Đảm bảo chống side-channel attack.
2. PQC Verifier Smart  Contract (On-chain PEP): Triển khai EVM và WASM để so sánh với nhau, triển khai ECDSA làm tiêu chuẩn cơ sở.
3. Off-chain Components (Prover/Relayer): Relayer/Attestor (off-chain verification/on-chain attestation), Prover (zk proof).

### 2.3 Invariants (khẳng định cần được kiểm chứng).

**I1. Integrity & Unforgeability:** Chống giả mạo giao dịch.
**I2. Cryptographic Correctness:** Đảm bảo logic xác thực hoạt động đúng, không xảy ra sai lệch hệ thống.
**I3. Cost Feasibility:** Chi phí gas phải đảm bảo trong ngưỡng nhất định.
**I4. Optimization Effectiveness:** Đảm bảo tính khả dụng trong các giải pháp tối ưu off-chain.
**I5. Reproducibility:** Các Metrics, Experiments phải có khả năng tái tạo nhất quán.


## 3) Crypto Solution
### 3.1 Crypto layer.
- Data in transit Protection: Dùng chữ ký số.
- Data at Rest Protection: Hash block.
- Signatures & Authentication: Base - ECDSA, PQC1 - CRYSALS-Dilithium, PQC2 - Falcon.
- Key Management: Client-side, khởi tạo và lưu trong môi trường dev/temporary.
### 3.2 AuthN layer (Authentication)
- User authentication: Sử dụng cơ chế thử thách - phản hồi (signature-based challenge-response). Với Challenge là transaction, và Response là PQC của transaction.
- Anti-replay (Không có session vì blockchain là phi trạng thái - stateless): Sử dụng cơ chế nonce để định danh giao dịch.

### 3.3 AuthZ layer (Authorization)
- Mô hình: PEP tại Smart contract.
- Thi hành: Deny-by-Default, Least Privilege.

## 4) Deployment Testing (Kế hoạch triển khai).
### D1. Xác thực trực tiếp trên EVM (EVM-based On-chain Naive Verification)
#### Stack: 
- **Blockchain:** Local Ethereum - HardHat.
- **Smart Contract:** Solidity.
- **Off-chain libs:**  PQClean/liboqs.
- **Client Scripting:** Ethers.js/Viem

#### Trọng tâm kiểm thử:
- Đo chi phí gas.
- Đo kích thước data.
### D2. Xác thực Trực tiếp trên WASM (WASM-based On-chain Verification)
#### Stack (Substrate local):
- **Blockchain:** Substrate local (Dev Node).
- **Smart Contract:** Rust to WASM = `!ink`.
- **Off-chain libs:**  pqcrypto.
- **Client Scripting:**  Polkadot.js.

#### Fallback stack (Cosmos):
- **Blockchain:** Wasmd.
- **Smart Contract:** Rust to WASM (framework: CosmWasm).
- **Off-chain libs:**  pqcrypto.
- **Client Scripting:**  CosmJS.

#### Trọng tâm kiểm thử:
- Đo chi phí gas.
- Đo kích thước data.
- So sánh với D1.

### D3. Xác thực Off-chain & Chứng thực On-chain (Attestation Model)
#### Stack:
- On-chain: tương tự D1.
- Off-chain: Relayer viết bằng Node.js hoặc Go.
#### Trọng tâm:
- Đo gas on-chain.
- Đo end-to-end latency.
- Kiểm tra trade-off giữa chi phí và độ tin cậy.

### D4. Xác thực với ZK-Proof (ZK-Proof Assisted Model) (*dự kiến*)
#### Stack
- On-chain: tương tự D1.
- Off-chain: Circom & snarkjs, mô phỏng arithmetic circuit của PQC, tạo proof.

#### Trọng tâm:
- Đo gas on-chain.
- Đo proofing-time off-chain.
- Đánh giá độ phức tạp triển khai.


## 5) Evaluation.
### E-Crypto
- **E-C1** Integrity & Forgery Resistance: Tỉ lệ false-accept = 0%, tỉ lệ false-reject = 0%.
- **E-C2** On-Chain Execution Cost: $\leq 15M$ gas.
- **E-C3** Transaction Data Cost: ECDSA x1, Falcon x12, Dilithium x40.

### E-AuthN
- **E-N1** Off-Chain Performance Cost: Độ trễ $\leq 5s$.

### E-AuthZ
- **E-Z1** Bussiness Logic Preservation: Tỉ lệ tương thích giữa ECDSA và PQC đạt 100%.

---
## Tham khảo [^1][^2][^3][^4][^5][^6][^7][^8][^9]:
[^1]: https://zknox.eth.limo/posts/2025/03/21/ETHFALCON.html
[^2]: https://eprint.iacr.org/2024/1287.pdf
[^3]: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.pdf
[^4]: https://blog.cloudflare.com/another-look-at-pq-signatures/
[^5]: https://pq-crystals.org/dilithium/data/dilithium-specification-round3.pdf
[^6]: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.pdf
[^7]: https://eprint.iacr.org/2017/279.pdf
[^8]: https://hackmd.io/@Giapppp/mlkem
[^9]: https://github.com/Tetration-Lab
