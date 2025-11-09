# Post-Quantum Cryptography - Digital Signature for Smart Contract.
> Nhan_laptop 
> 2 Mô hình xuyên suốt trong blog: Dilithium & Falcon.
> This blog will be written in Vietnamese.
## Smart Contract: 
Smart Contract - chương trình tự thực thi trên các hệ thống blockchain, tự động thực hiện các giao dịch khi có đủ các điều kiện cần thiết, xóa bỏ sự cần thiết của một bên thứ ba.
Lần đầu tiên được đề xuất bởi Nick Szabo vào năm 1994, Smart contracts đã đóng vai trò quan trọng trong nhiều ứng dụng, bao gồm bất động sản, thương  mại và chuỗi cung ứng. 
Smart contract hợp lí hóa quy trình bằng cách tự động thực hiện các thỏa thuận giữa các bên, nhưng mối liên hệ của chúng với các hành động thực tế - như vận chuyển hàng hóa - vẫn đang được phát triển.
Lợi thế cốt lõi của smart contracts là bỏ đi sự tham gia của bên thứ 3, tuy nhiên với công nghệ này thì vẫn phải đối mặt một số thách thức và sự hạn chế cần được giải quyết.
## Post - Quantum Cryptography: Modern Digital Signature for smart contract. 


**Introduction**:


Với sự phát triển công nghệ hiện nay, một thuật toán chữ ký mới là cần thiết, bởi vì với các thuật toán hiện hành dựa trên độ khó của bài toán phân tích thừa số nguyên tố và bài toán lo-ga-rit rời rạc sẽ bị giải bởi máy tính lượng tử. 
Có nhiều sơ đồ chữ ký điện thử đã được giới thiệu:
- LMS/XMSS (SP 800 - 208) - Hash-based signature schemes được đánh giá là khởi tạo và ký với kích thước khóa công khai nhỏ, nhưng chữ ký có kích thước lớn. 
- SLH-DSA (FIPS 205, SPHINCS+) - Hash-based signature scheme có sự tăng thời gian tuyến tính trong lúc xác thực - nhanh khi kích thước public key nhỏ và ngược lại. 
- ML-DSA (FIPS 204, Dilithium) - Lattice-based signature scheme nhanh trong ký và xác thực, nhưng kích thước của public key và signature lại lớn. Dilithium cung cấp tốt sự cân bằng giữa thời gian ký/xác thực và kích thước public key/signature, mặc dù kích thước lớn  trong key và signature sẽ làm tăng phí lưu trữ for transprency log operator.
- Falcon - Lattice-based signature scheme nhanh trong xác thực, với kích thước lớn trong public key and signature, tuy nhiên nhỏ hơn so với Dilithium, và ký chậm hơn Dilithium. Như Dilithium, Falcon cung cấp tốt sự cân bằng giữa signing/verification time and public key/signature sizes. Tuy nhiên, Falcon đôi phần sẽ phức tạp trong cài đặt và một vài vấn đề xoay quanh nó khi sử dụng floating point operations.

### ML-DSA (FIPS 204, Dilithium).

Dilithium - sơ đồ ký điện tử mạnh mẽ under CPA dựa trên độ khó của bài toán lattice (CVP - SVP) over module lattices. Định nghĩa bảo mật an toàn có nghĩa rằng một Adversary có quyền truy cập vào a signing oracle nhưng không thể tạo một chữ ký của message của một ai đó mà anh ta chưa từng nhìn thấy từ trước, hoặc không thể tạo ra một chữ kí khác của một message mà anh ta đã thấy ký từ trước. Dilitium là một trong những thuật toán ứng cử được gửi đến [NIST post-quantum cryptography project](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography).

#### Scientific Background 
Thiết kế của Dilithium được dựa trên "[Fiat-Shamir with Aborts](https://link.springer.com/chapter/10.1007/978-3-642-10366-7_35)" kĩ thuật của Lyubashevsjt sử dụng các rejection sampling để làm cho lattice-based Fiat-Shamir schemes compact and secure. Sơ đồ với kích thước signature nhỏ nhất sử dụng cách tiếp cận này là của  [Ducas, Durmus, Lepoint, and Lyubashevsky](https://eprint.iacr.org/2013/383), phương pháp dựa trên NTRU assumption và chủ yếu sử dụng Gaussian sampling để tạo signatures. Bởi vì Gaussian sampling khó cài đặt an toàn và hiệu quả, chúng ta chọn sử dụng phân phối đều. Dilithium cải tiến trên sơ đồ hiệu quả nhất mà chỉ sử dụng uniform distribution, do [Bai and Galbraith](https://eprint.iacr.org/2013/838), bằng cách sử dụng một kĩ thuạt mới đó là shrinks the public key gấp 2 lần. Tóm lại, Dilithium là sơ đồ chữ ký điện tử có kích thước public key và signature nhỏ nhất trong số các lattice-based signature scheme mà chỉ dựa trên uniform sampling. 

#### Computational Assumptions. 

An toàn cho lattice-based signature schemes là thường liên quan đến Learning With Errors (LWE) problem và short integer solution (SIS) problem. The LWE problem yêu cầu chúng ta recover một vector s-secret $\in \mathbb{Z}^n_q$ từ một tập random "noisy" linear system over $\mathbb{Z}_q$ cho trước của một định dạng $A*t=0$ với $||t||_\infty$ nhỏ. Với sự lựa chọn parameters hợp lí, những bài toán như trên là khó trong những kĩ thuật tốt nhất hiện tại kể cả Gaussian elimination. 

Khi module $\mathbb{Z}^n_q$ trong LWE và SIS được thay thế bằng một module over ring later than $\mathbb{Z}_q$(e.g,$\mathbb{R}_q$), vấn đề này được gọi là Module Learning With Errors (MLWE) and Module Short Intefer Solution (MSIS). Vấn đề bảo mật của ML-DSA dựa trên bài toán MLWE problem over $\mathbb{R}_q$ và biến thể không chuẩn của MSIS được gọi là SelfTargetMSIS. 

#### ML-DSA Construction

ML-DSA là một Schnorr-like signature với một vài tối ưu hóa. The Schnorr signature scheme ứng dụng Fiat-Shamir heuristic cho một interactive protocol giữa người xác thực biết `g` (Số sinh của một nhóm mà nơi đó discrete rời rạc được biết là khó) và giá trị $y = g^x$ và một prover người biết g và x. Giao thức tương tác, nơi mà prover chứng minh kiến thức về x cho người xác thực, bao gồm 3 bước: 
- Commitment: Prover tạo ra một số nguyên dương `r` bé hơn order của `g` và commit giá trị của nó bằng cách gửi $g^r$ cho người xác thực. 
- Challenge: Người xác thực gửi một số nguyên dương c bé hơn bậc g cho Prover. 
- Response: Prover trả vể giá `s = r - cx` giảm modulo bậc của `g`, và người xác thực kiểm tra xem $g^s .y^c = g^r$.

Giao thức này được thực hiện không tương tác và chuyển thành một signature scheme bằng cách thay thế `Verifier's random choice of c` ở bước Challenge với một quá trình xác định đó là `pseudorandomly` với giá trị đầu `c` từ một `hash` của commitment $g^r$ được nối liên- `|` với messages để được ký. Với signature scheme này, `y` là một public key, `x` là private key.
Ý tưởng cốt lõi của $ML-DSA$ và các mỗ hình $Lattice\  signature$ liên quan được xây dựng bằng lược đồ chữ ký từ `analogous interactive protocol`, nơi mà Prover biết được matrices $A \in \mathbb{Z}_q^{K\mathbb{x}L}$, $S_1 \in \mathbb{Z}_q^{L\mathbb{x}n}$, và $S_2 \in \mathbb{Z}_q^{K\mathbb{x}n}$ với hệ số nhỏ ( cho $S_1,\ S_2$) chứng minh hiểu biết về các ma trận với Verifier người biết $A$ và $T \in \mathbb{Z}_q^{K\mathbb{x}n}= AS_1+S_2$. Ví dụ một `interactive protocol` sẽ tiến hành như sau: 

1. Commitment: Prover tạo ra $y\in \mathbb{Z}^L_q$ voiws với hệ số nhỏ và commits to its value bằng cách gửi $w_{approx} = Ay +y_2$ tới Verifier, với $y_2\in \mathbb{Z}^K_q$ là một vector với hệ số nhỏ.
2. Challenge: Verifier gửi một vector $c\in \mathbb{Z}^n_q$ với hệ số nhỏ tới Prover. 
3. Response: Prover trả về giá trị $z=y+S_1c$, và Verifier kiểm tra $z$ có hệ số nhỏ và $Az-Tc\approx{w_{approx}}$.

Như đã viết, giao thức trên có một lỗ hổng bảo mật: Response z sẽ bị biased một các trực tiếp đến với giá trị $S_1$. Tương tự $r=w_{approx}-Az+Tc = y_2 + S_2c$ bị biased một cách trực tiếp đến giá trị $S_2$. Tuy nhiên. lỗ hổng này có thể được sửa khi chuyển đổi `the interative protocol` sang `a signature scheme`. Như với Schnorr signatures, người ký lấy được thử thách bằng `pseudorandom process` từ một hàm hash của `commitment concatenated and message`. Để sửa lỗi bias, Người ký áp dụng lấy mẫu từ chối $z$; nếu hệ số của $z$ vượt khỏi khoảng giá trị mặc định, quá trình ký sẽ bị hủy bỏ, người ký sẽ bắt đầu lại với giá trị y mới. Tương tự mẫu từ chối phải được áp dụng cho $r$. Trong Fiat-Shamir với Aborts signature được đơn giản hóa, Public key là $(A,T)$ và private key $(S_1,S_2)$.

Trong tiêu chuẩn ML-DSA, một loạt các điều chỉnh và sửa đổi được thêm vào khuôn mẫu cở bản vì vấn đề an toàn và hiệu quả: 
- Để giảm kích cỡ key và signature và sử dụng phép nhân đa thức nhanh dựa trên NTT-based polynomial, ML-DSA sử dụng ma trận module-structured. Liên quan đến sơ đồ cơ bản được mô tả ở trên, thì ở lược đồ này thay thế khối n.n chiều của một ma trận và khối n chiều của một vector với các đa thức trong vòng-vành $R_q$. Vì vậy, thay vì $A\in\mathbb{Z}_q^{K\mathbb{x}L}$, $T\in\mathbb{Z}_q^{K\mathbb{x}L}$, $S_1\in\mathbb{Z}_q^{L\mathbb{x}n}$, $S_2\in\mathbb{Z}_q^{L\mathbb{x}n}$, $y\in\mathbb{Z}_q^{L}$, $c\in\mathbb{Z}_q^{n}$, thì ML-DSA là $A\in\mathbb{R}_q^{k\times l}$, $t\in\mathbb{R}_q^{k}$, $s_1\in\mathbb{R}_q^{k}$, $s_2\in\mathbb{R}_q^{l}$, $c\in\mathbb{R}_q$, với $l=L/n$ và $k=K/n$.
- Hơn thế nữa để giảm kích thước public key, ma trận $A$ được lấy ngẫu nhiên từ 256-bit công khai `p`, giá trị được bao gồm trong ML-DSA public key trong $A$.
- Để giảm kích thước khóa công khai hơn nữa, ML-DSA public key thay thế một giá trị nén $t_1$ cho $t$, với giảm bớt $d$ các bit có thứ tự thấp mỗi hệ số.
- Để đạt được tính chất  không thể làm giả (BUFF), ML-DSA không trực tiếp ký lên message $M$ nhưng sẽ kí vào một thông điệp đại diện $u$ - có được bằng các hashing phép nối giữa một hash của public key và $M$.
- Để giảm kích thước của signature thay vì bao gồm cam kết $w_\text{Approx}= Ay+y_2$ trong signature, ML-DSA signature sử dụng phiên bản làm trọng $w = Ay$ như cam kết $w_1$ và chỉ bao gồm băm $\hat{c}$ của $w_1||u$.
- Để đảm bảo rằng $w_1$ có thể tái tạo được bởi người xác thực bởi $z$ và  giá trị nén $t_1$, chữ kĩ cũng phải bao gồm một hint $h\in R_2^K$ được tính bởi người ký bằng cách sử dụng khó riêng của người ký.
- Thêm nữa, để đảm bảo tính đúng đắn, phải bao gồm giai đoạn loại bỏ thứ 2.

## References [^1]:
[^1]: https://nvlpubs.nist.gov/nistpubs/fips/nist.fips.204.pdf
