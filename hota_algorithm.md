
# HOTA Metric — Compact Algorithm + Variable Reference

<table>
<tr>
<td valign="top" width="58%">

## Compact Paper-Style Algorithm

### Per Video / Sequence

**Input**

\[
\{\mathcal{GT}_t,\mathcal{Pred}_t,S^t\}_{t=1}^{T},
\qquad
\mathcal{A}=\{0.05,0.10,\ldots,0.95\}
\]

Total G  groundtruth tracks and P prediction tracks

---

### 1. Similarity normalization

For frame \(t\), ground-truth detection \(i\), and prediction \(j\):

\[
R^t_{ij}
=
\frac{
S^t_{ij}
}{
\sum_{i=1}^{N_t} S^t_{ij}
+
\sum_{j=1}^{M_t} S^t_{ij}
-
S^t_{ij}
}
\]

Shape of \(R^t\) : \(N_t\times M_t\)

<!-- 
If the denominator is zero, set

\[
R^t_{ij}=0.
\] -->

---

### 2. Accumulate global identity co-occurrence

For every GT identity \(g\) and predicted identity \(p\):

\[
C_{gp}
=
\sum_{
t,i,j:
ID_G(i)=g,\;
ID_P(j)=p
}
R^t_{ij}.
\]

Also count detections belonging to each identity over the whole video:

\[
N^G_g
=
\sum_t
\#\{
i:ID_G(i)=g
\},
\]

\[
N^P_p
=
\sum_t
\#\{
j:ID_P(j)=p
\}.
\]

Shape of C : \(G \times P\)

---

### 3. Global identity alignment

\[
\boxed{
A_{gp}
=
\frac{
C_{gp}
}{
N^G_g+N^P_p-C_{gp}
}
}
\]  

Shape of A : \(G \times P\)


This score measures how strongly GT identity \(g\) aligns with predicted identity \(p\) over the complete video.

---

### 4. Frame-level matching score

For every pair \((i,j)\) in frame \(t\):

\[
\boxed{
W^t_{ij}
=
A_{\,ID_G(i),\,ID_P(j)}
\cdot
S^t_{ij}
}
\]

---

### 5. Optimal one-to-one matching

Apply Hungarian matching:

\[
\boxed{
\Pi_t
=
\arg\max_{\Pi}
\sum_{(i,j)\in\Pi}
W^t_{ij}
}
\]

subject to every GT detection and prediction being used at most once.

---

### 6. Threshold accepted matches

For every threshold \(\alpha\in\mathcal{A}\):

\[
\boxed{
\Pi_t^\alpha
=
\{
(i,j)\in\Pi_t
:
S^t_{ij}\ge\alpha
\}
}
\]

Only Hungarian matches satisfying the original localization threshold are retained.

---

### 7. Detection counts

\[
\boxed{
TP_\alpha
=
\sum_t
|\Pi_t^\alpha|
}
\]

\[
\boxed{
FN_\alpha
=
\sum_t
\left(
N_t-|\Pi_t^\alpha|
\right)
}
\]

\[
\boxed{
FP_\alpha
=
\sum_t
\left(
M_t-|\Pi_t^\alpha|
\right)
}
\]

---

### 8. Identity-pair match counts

For every GT identity \(g\) and predicted identity \(p\):

\[
\boxed{
M^\alpha_{gp}
=
\sum_{t=1}^{T}
\#
\left\{
(i,j)\in\Pi_t^\alpha
:
ID_G(i)=g,\;
ID_P(j)=p
\right\}
}
\]

Equivalently,

\[
M^\alpha_{gp}
=
\text{number of accepted detection matches across the video}
\]

whose GT detection belongs to identity \(g\) and whose prediction belongs to identity \(p\).

Thus,

\[
M^\alpha_{gp}=3
\]

means GT track \(g\) and predicted track \(p\) were accepted as a matched pair in **three frames** at threshold \(\alpha\).

---

### 9. Association Accuracy

For an identity pair \((g,p)\):

\[
AssIoU^\alpha_{gp}
=
\frac{
M^\alpha_{gp}
}{
N^G_g+N^P_p-M^\alpha_{gp}
}
\]

Then

\[
\boxed{
AssA_\alpha
=
\frac{
\displaystyle
\sum_{g,p}
M^\alpha_{gp}
\,
AssIoU^\alpha_{gp}
}{
\max(1,TP_\alpha)
}
}
\]

or directly,

\[
AssA_\alpha
=
\frac{
\displaystyle
\sum_{g,p}
M^\alpha_{gp}
\frac{
M^\alpha_{gp}
}{
N^G_g+N^P_p-M^\alpha_{gp}
}
}{
\max(1,TP_\alpha)
}.
\]

---

### 10. Localization Accuracy

Accumulate localization similarity of accepted matches:

\[
L_\alpha
=
\sum_t
\sum_{(i,j)\in\Pi_t^\alpha}
S^t_{ij}.
\]

Then

\[
\boxed{
LocA_\alpha
=
\frac{
\max(10^{-10},L_\alpha)
}{
\max(10^{-10},TP_\alpha)
}
}
\]

If TP=0, then LocA = 0

---

### 11. Association Recall and Precision

\[
\boxed{
AssRe_\alpha
=
\frac{
\displaystyle
\sum_{g,p}
M^\alpha_{gp}
\frac{M^\alpha_{gp}}{N^G_g}
}{
\max(1,TP_\alpha)
}
}
\]

\[
\boxed{
AssPr_\alpha
=
\frac{
\displaystyle
\sum_{g,p}
M^\alpha_{gp}
\frac{M^\alpha_{gp}}{N^P_p}
}{
\max(1,TP_\alpha)
}
}
\]

---

### 12. Detection metrics

\[
\boxed{
DetRe_\alpha
=
\frac{
TP_\alpha
}{
TP_\alpha+FN_\alpha
}
}
\]

\[
\boxed{
DetPr_\alpha
=
\frac{
TP_\alpha
}{
TP_\alpha+FP_\alpha
}
}
\]

\[
\boxed{
DetA_\alpha
=
\frac{
TP_\alpha
}{
TP_\alpha+FN_\alpha+FP_\alpha
}
}
\]

---

### 13. HOTA

\[
\boxed{
HOTA_\alpha
=
\sqrt{
DetA_\alpha
\cdot
AssA_\alpha
}
}
\]


---

## Combine Multiple Videos / Sequences

Let the dataset contain

\[
V_1,V_2,\ldots,V_K.
\]

### 1. Sum detection counts

\[
TP_\alpha
=
\sum_{k=1}^{K}
TP_\alpha^{(k)}
\]

\[
FN_\alpha
=
\sum_{k=1}^{K}
FN_\alpha^{(k)}
\]

\[
FP_\alpha
=
\sum_{k=1}^{K}
FP_\alpha^{(k)}.
\]

---

### 2. TP-weighted association metrics

For

\[
X\in\{AssA,AssRe,AssPr\},
\]

\[
\boxed{
X_\alpha
=
\frac{
\displaystyle
\sum_{k=1}^{K}
TP_\alpha^{(k)}
X_\alpha^{(k)}
}{
TP_\alpha
}
}
\]

---

### 3. TP-weighted localization accuracy

\[
\boxed{
LocA_\alpha
=
\frac{
\displaystyle
\sum_{k=1}^{K}
TP_\alpha^{(k)}
LocA_\alpha^{(k)}
}{
TP_\alpha
}
}
\]

---

### 4. Recompute dataset detection metrics

\[
DetRe_\alpha
=
\frac{
TP_\alpha
}{
TP_\alpha+FN_\alpha
}
\]

\[
DetPr_\alpha
=
\frac{
TP_\alpha
}{
TP_\alpha+FP_\alpha
}
\]

\[
DetA_\alpha
=
\frac{
TP_\alpha
}{
TP_\alpha+FN_\alpha+FP_\alpha
}
\]

---

### 5. Recompute dataset HOTA

\[
\boxed{
HOTA_\alpha
=
\sqrt{
DetA_\alpha
\cdot
AssA_\alpha
}
}
\]

Also:

\[
OWTA_\alpha
=
\sqrt{
DetRe_\alpha
\cdot
AssA_\alpha
}.
\]    

OWTA score focuses only on matched detections by not inluding FP's 


---

### Final reported HOTA

\[
\boxed{
HOTA
=
\frac{1}{19}
\sum_{\alpha=0.05}^{0.95}
HOTA_\alpha
}
\]

where \(\alpha\) increases in steps of \(0.05\).

</td>

<td valign="top" width="42%">














## 

### Per-frame quantities

| Variable | Meaning | Shape |
|---|---|---|
| \(\mathcal{GT}_t\) | GT detections in frame \(t\) | set/list of size \(G_t\) |
| \(\mathcal{Pred}_t\) | Predicted detections in frame \(t\) | set/list of size \(P_t\) |
| \(S^t\) | Localization similarity matrix | \(G_t\times P_t\) |
| \(T\) | Number of frames in the video | scalar |
| \(N_t\) | Number of GT detections in frame \(t\) | scalar |
| \(M_t\) | Number of predicted detections in frame \(t\) | scalar |
| \(S^t_{ij}\) | Similarity between GT detection \(i\) and prediction \(j\) | scalar |
| \(ID_G(i)\) | GT track identity of detection \(i\) | scalar ID |
| \(ID_P(j)\) | Predicted track identity of detection \(j\) | scalar ID |

---

### Whole-video identity quantities

| Variable | Meaning | Shape |
|---|---|---|
| \(G\) | Number of unique GT identities | scalar |
| \(P\) | Number of unique predicted identities | scalar |
| \(N^G\) | Total detection count for every GT identity | \(G\) |
| \(N^G_g\) | Number of GT detections belonging to identity \(g\) across the video | scalar |
| \(N^P\) | Total detection count for every predicted identity | \(P\) |
| \(N^P_p\) | Number of predictions belonging to identity \(p\) across the video | scalar |
| \(A\) | Global identity alignment matrix | \(G\times P\) |
| \(A_{gp}\) | Alignment score between GT identity \(g\) and predicted identity \(p\) | scalar |

---

### Matching quantities

| Variable | Meaning | Shape |
|---|---|---|
| \(W^t\) | HOTA matching-score matrix | \(N_t\times M_t\) |
| \(W^t_{ij}\) | Matching score \(A_{gp}S^t_{ij}\) | scalar |
| \(\Pi_t\) | Hungarian one-to-one matches in frame \(t\) | set of pairs |
| \(\Pi_t^\alpha\) | Hungarian matches accepted after threshold \(\alpha\) | set of pairs |
| \(\alpha\) | Localization threshold | scalar |
| \(\mathcal A\) | Set of evaluation thresholds | 19 scalars |

---

### Identity-pair match counts

| Variable | Meaning | Shape |
|---|---|---|
| \(M^\alpha\) | Accepted identity-pair match-count matrix | \(G\times P\) |
| \(M^\alpha_{gp}\) | Number of accepted detection matches between GT identity \(g\) and prediction identity \(p\) across all frames | scalar |

**Intuition**

If

\[
M^\alpha_{gp}=5,
\]

then five accepted detection pairs across the video connected GT identity \(g\) with predicted identity \(p\) at threshold \(\alpha\).

This quantity is therefore the bridge between **frame-level detection matching** and **track-level association quality**.

---

### Per-threshold detection counts

| Variable | Meaning | Shape |
|---|---|---|
| \(TP_\alpha\) | Total accepted matched detections | scalar |
| \(FN_\alpha\) | Total unmatched GT detections | scalar |
| \(FP_\alpha\) | Total unmatched predicted detections | scalar |
| \(L_\alpha\) | Sum of localization similarities of accepted matches | scalar |

---

### Per-threshold metrics

| Variable | Meaning | Shape |
|---|---|---|
| \(AssIoU^\alpha_{gp}\) | Association IoU/Jaccard for identity pair \((g,p)\) | scalar |
| \(AssA_\alpha\) | Association Accuracy | scalar |
| \(AssRe_\alpha\) | Association Recall | scalar |
| \(AssPr_\alpha\) | Association Precision | scalar |
| \(LocA_\alpha\) | Localization Accuracy | scalar |
| \(DetRe_\alpha\) | Detection Recall | scalar |
| \(DetPr_\alpha\) | Detection Precision | scalar |
| \(DetA_\alpha\) | Detection Accuracy | scalar |
| \(HOTA_\alpha\) | HOTA at threshold \(\alpha\) | scalar |

---

### Multi-video quantities

| Variable | Meaning | Shape |
|---|---|---|
| \(K\) | Number of videos/sequences | scalar |
| \(k\) | Video index | scalar |
| \(TP_\alpha^{(k)}\) | TP count from video \(k\) | scalar |
| \(FN_\alpha^{(k)}\) | FN count from video \(k\) | scalar |
| \(FP_\alpha^{(k)}\) | FP count from video \(k\) | scalar |
| \(AssA_\alpha^{(k)}\) | Association Accuracy of video \(k\) | scalar |
| \(AssRe_\alpha^{(k)}\) | Association Recall of video \(k\) | scalar |
| \(AssPr_\alpha^{(k)}\) | Association Precision of video \(k\) | scalar |
| \(LocA_\alpha^{(k)}\) | Localization Accuracy of video \(k\) | scalar |

</td>
</tr>
</table>
