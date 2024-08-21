# Transformer-based Korean-English translation model (2022.11.)
[나동빈님의 코드](https://github.com/ndb796/Deep-Learning-Paper-Review-and-Practice/blob/master/code_practices/Attention_is_All_You_Need_Tutorial_\\(German_English\\).ipynb) 를 참고하여 한-영 번역 모델을 구현했습니다.  
데이터는 AI Hub의 한국어-영어 병렬 말뭉치 중 1_구어체(1).xlsx 파일을 사용했으며 20만 개의 병렬 데이터로 구성되어 있습니다.

## 배경
Attention is all you need 논문에서는 Adam Optimizer를 사용했지만 Adam은 Transformer와 같이 일부 깊은 네트워크에서 학습이 어렵다는 단점을 가집니다. 따라서 논문에서는 Warm-up and decay 메커니즘을 함께 적용했습니다. 초기 warm-up 구간에는 Learning Rate를 작게 설정하여 불안정한 모멘텀을 갖는 것을 방지하는 것입니다. 이후에는 Warm-up 구간을 튜닝할 필요 없는 Rectified Adam(Liu et al., 2020)이라는 Optimizer가 제기되었으며, Warm-up and decay 메커니즘조차 필요없이 Normalization 위치만 변경하는 방법론이 On Layer Normalization in the Transformer Architecture (Xiong et al., 2020)에서 제기되었습니다.

## 실험
본 코드에서는 Post-LN + Adam, Post-LN + RAdam, Pre-LN + Adam 3가지의 성능을 비교하는 실험을 진행했습니다.  
전처리는 Subword Segmentation없이 토큰화만 진행했습니다. 한국어는 Mecab, 영어는 Spacy를 사용했습니다.  
자세한 과정은 Transformer_nmt.ipynb 파일을 참고해주세요.

## 결과
번역 모델에서는 성능 평가 지표로 BLEU 스코어를 사용합니다. 낮은 Perplexity가 좋은 번역 품질을 보장하지 못하기 때문입니다. 특히 한국어는 교착어에 속하기 때문에 어순이 바뀌어도 무방하지만 Perplexity 값은 달라지게 됩니다. 따라서 N-gram을 활용한 BLEU 스코어를 활용하는데 corpus_bleu(trgs, pred_trgs, weights=(1, 0, 0, 0)) 함수에서 weights 첫 번째 값을 1로 두면 unigram을 사용한다는 뜻입니다. 표준은 0.25,0.25,0.25,0.25 로 가중치를 1-gram, 2-gram, 3-gram, 4-gram 동일하게 두는 값을 사용합니다.

Attention is all you need 논문에서의 번역기 성능은 20후반대 이상을 기록했습니다. 50,000개 이하의 데이터를 사용할 때는 표준 BLEU 값을 계산하는 것이 무의미할 정도로 낮은 점수를 기록했으며 20만 개 이상부터 유의미한 값이 산출되었습니다. Layer Norm의 위치를 바꾸는 것만으로 성능이 크게 향상되었으며 RAdam을 사용할 때는 기대와 달리 큰 성능 향상은 없었습니다.
