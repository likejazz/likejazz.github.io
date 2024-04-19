---
layout: post
title: 코사인 유사도의 의미
tags: ["Machine Learning"]
last_modified_at: 2018/06/01 00:00:00
---

<div class="message">
일반적으로 문서간의 유사도를 비교할 때는 코사인 유사도<sup>cosine similarity</sup>를 주로 사용한다. 여기서는 그 동안 습관적으로 사용해오던 코사인 유사도의 의미를 수식과 함께 예제를 통해 살펴보도록 한다.
</div>

<small>
*2018년 6월 1일 초안 작성*  
</small>

<!-- TOC -->

- [본론](#본론)
    - [데이터 준비](#데이터-준비)
    - [메트릭 선별](#메트릭-선별)
        - [유클리드 거리](#유클리드-거리)
        - [코사인 유사도](#코사인-유사도)
    - [무슨 일이 일어 났을까](#무슨-일이-일어-났을까)
    - [언제 코사인 유사도를 사용하는가](#언제-코사인-유사도를-사용하는가)
        - [코사인 유사도 예제](#코사인-유사도-예제)
        - [트위터 분류](#트위터-분류)
- [참고](#참고)

<!-- /TOC -->

## 본론
여기서 사용한 주피터 코드와 설명은 [Euclidean vs. Cosine Distance](https://cmry.github.io/notes/euclidean-v-cosine)에서 가져왔다. 코사인 유사도의 의미에 대해 매우 깔끔하게 잘 정리한 글이다. 작성자는 강의 중 "언제 코사인 유사도를 사용하나요?"라는 학생의 질문을 받고 글을 작성했다고 한다.

### 데이터 준비
먼저 아래와 같은 데이터를 준비하고 시각화 해본다.

```python
X = np.array([[6.6, 6.2, 1],
              [9.7, 9.9, 2],
              [8.0, 8.3, 2],
              [6.3, 5.4, 1],
              [1.3, 2.7, 0],
              [2.3, 3.1, 0],
              [6.6, 6.0, 1],
              [6.5, 6.4, 1],
              [6.3, 5.8, 1],
              [9.5, 9.9, 2],
              [8.9, 8.9, 2],
              [8.7, 9.5, 2],
              [2.5, 3.8, 0],
              [2.0, 3.1, 0],
              [1.3, 1.3, 0]])

df = pd.DataFrame(X, columns=['weight', 'length', 'label'])

ax = df[df['label'] == 0].plot.scatter(x='weight', y='length', c='blue', label='young')
ax = df[df['label'] == 1].plot.scatter(x='weight', y='length', c='orange', label='mid', ax=ax)
ax = df[df['label'] == 2].plot.scatter(x='weight', y='length', c='red', label='adult', ax=ax)
ax
```

![](data:image/png;base64,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)

키와 몸무게에 따른 나이를 추측하는 가상의 데이터로 young, mid, adult 세 가지 클래스는 두 가지 피쳐로 매우 잘 구분된다. k-NN을 적용한다고 가정할때 어떤 거리 메트릭<sup>distance metric</sup>을 사용하는 것이 적절한지 살펴보자.

### 메트릭 선별
0, 1, 4번 인스턴스를 선별하여 14번 인스턴스에 어떤 레이블을 부여하는게 적절한지 살펴보자.

```python
df2 = pd.DataFrame([df.iloc[0], df.iloc[1], df.iloc[4]], columns=['weight', 'length', 'label'])
df3 = pd.DataFrame([df.iloc[14]], columns=['weight', 'length', 'label'])

ax = df2[df2['label'] == 0].plot.scatter(x='weight', y='length', c='blue', label='young')
ax = df2[df2['label'] == 1].plot.scatter(x='weight', y='length', c='orange', label='mid', ax=ax)
ax = df2[df2['label'] == 2].plot.scatter(x='weight', y='length', c='red', label='adult', ax=ax)
ax = df3.plot.scatter(x='weight', y='length', c='gray', label='?', ax=ax)
ax
```

![](data:image/png;base64,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%0A7dZA7969ufXWWzl06BDl5eWMHz+e66+/ntzc3IRsr6ysjEmTJnHixAm6dOnCb3/7WyC4oOy6666j%0AX79+CdmuiJx9LAiN9q2goMAb36ry7bff5pJLLgmpoo5N361Ix2BmG9y9oLnlInGwWERETk1BICIS%0AcQoCEZGIUxCIiEScgqCNPfXUU+Tm5pKbm8tTTz1VN+6taEO9dOnSugvSGuvWrRsQNJe7+uqr4yta%0ARCItUqePJtr+/fuZNWsWJSUlmBlDhw5lwoQJ9OzZs1VtqCdMmMCECRNOu0xmZia9e/dm7dq1DB8+%0APN4/QUQiKFozgqoK2FccPMapqTbUjz32GGPHjuXcc8+lZ8+ejB07lmXLlgEN21CXlpYyYMAAbr75%0AZvr168dNN93Eq6++yvDhw8nNzeWNN94AYMGCBXU3t9m5cyeXX345+fn53H///Q1qUVtpEYlHdIKg%0AdBEs6QsrxwaPpfH1oW6qDXXXrl256KKL6pbJyspi165dQMM21ADvvfced999Nzt27GDHjh08++yz%0ArFmzhjlz5vDggw+etL0777yT2267jS1bttC7d+8G7xUUFLB69eq4/h4Ria5oBEFVBawvgpojUH0w%0AeFxfFPfMYMaMGSxfvpySkhLuueee0y5bvw01QE5ODvn5+XTq1ImBAwcyevRozIz8/HxKm7gb0dq1%0Aa5k8eTIA3/ve9xq8d/7557N79+64/hYRia5oBEFlKXRqdPu3Tp2D8TjUtqE+fPgwVVVV9OnThw8+%0A+KDu/fLycvr06QOc3Eq6tnU0QKdOneped+rU6ZStpM2syfGqqiq6du0a198iItEVjSBIz4YTjfpQ%0An6gOxuPQuA31VVddVdfy+eOPP+aVV16pazHduA31mRo+fDjPPfccwEnHA9555x3y8vJa/4eISKRF%0AIwjSMuGr8yGlK3TuETx+dX4w3kr121Dfd999FBcXs2nTJh544AGGDRvGsGHDmDFjBueeey7wWRvq%0A1nrkkUd47LHHyM/PrzvuUEttpUUkHtFqOldVEewOSs+OKwRaY8+ePUydOpXly5e3+bpHjhzJkiVL%0AGtyOMh5qOifSMbS06Vy0riNIy0x6ANSq34b6TK4laE5FRQU//vGP2ywERCR6ohUEIZs0aVKbrzMz%0AM5PrrruuzdcrItERjWMEIiJySgoCEZGIUxCIiEScgkBEJOIUBG3s6quvJiMjg/Hjx5/03sSJE8/o%0AorJVq1Y1uZ7GaltSl5aW8uyzz9aNb9myhZtvvrnF2xORaFIQtLHp06fzzDPPnDS+bds2ampq+NKX%0AvpSwbTcOgvz8fMrLyykrK0vYNkXk7BetIKiogOLi4DFOTbWh3rp1K6NHj27QXK5W/TbUALfddhsF%0ABQUMHDiQmTNn1o0vW7aMAQMGMGTIEBYvXlw3/vOf/5w5c+bUvc7LyzupOd19993H6tWr+fKXv8xv%0AfvMbAL71rW/VtaYQEWlKdIJg0SLo2xfGjg0eF7V9G+rT9ftp3Ib6l7/8JSUlJWzevJm//vWvbN68%0AmaqqKm699VZeeuklNmzYwIcffnhGNc2ePZsRI0awadMm7rrrLkAtqkWkedEIgooKKCqCI0fg4MHg%0Asago7pnBmbSh3rNnD5mZn13V/MILLzBkyBAGDx7Mtm3b2L59Ozt27CAnJ4fc3FzMjClTpsRVH6hF%0AtYg0LxpBUFoKXRq1oe7cORiPQ+M21KfTtWvXumV27tzJnDlzWLFiBZs3b2bcuHHNfr5xG+vmlq+/%0AnFpUi8jpRCMIsrPhWKM21NXVwXgcGrehPp1LLrmE9957D4BDhw6Rnp7O5z//efbu3cuf/vQnAAYM%0AGEBpaSl///vfAVhUb/dVdnY2GzduBGDjxo3s3LnzpG10796dw4cPNxhTi2oRaU40giAzE+bPh65d%0AoUeP4HH+/GC8lZpqQ71y5UpGjBjBd77zHVasWEFWVhZ//vOfgYZtqAcNGsTgwYMZMGAAN954Y91N%0A59PS0pg3bx7jxo1jyJAhnH/++XXb+/a3v83+/fsZOHAgc+fOpV+/fifVdNlll5GSksKgQYPqDhar%0ARbWINCdabagrKoLdQdnZcYVAaxw5coQrr7yStWvXkpKSkpRtHj16lCuuuII1a9aQmtry/oJqQy3S%0AMagNdVMyM5MeALW6du3KrFmz2LVrF1/84heTss2ysjJmz559RiEgItGjX4gkqr1tZbLk5uaSm5ub%0A1G2KyNknlGMEZnaXmW0zs61mtsjM0sKoQ0REQggCM+sD3AEUuHsekAJ8N9l1iIhIIKyzhlKBrmaW%0ACnwO0BVPIiIhSXoQuPsuYA5QBuwBDrr7K42XM7MfmFmJmZVUtEFvIBERaVoYu4Z6AoVADvAFIN3M%0ATuql4O7z3L3A3QsyQzrTpzVO14Ya4I477qhrG13r4Ycf5umnnwZg2rRp5OXl0b9/f1566SUAXn75%0AZWbMmJHYwkUkssLYNTQG2OnuFe5eDSwGvh5CHQlxqjbUACUlJXz88ccNxo4fP86TTz7JjTfeCMAN%0AN9zA1q1bWbp0aV3juHHjxvHSSy/x6aefJrZ4EYmkMIKgDPiamX3OzAwYDbydjA1XVlaya9cuKisr%0A417XmbahrqmpYfr06Tz00EMNxleuXMmQIUPqzvW/9tprgeBisLS04GQqM+Mb3/gGL7/8ctx1i4g0%0AlvTrCNx9vZn9HtgIHAfeBOYlertbtmxh6dKlpKSkUFNTQ2FhYVw9eOq3oT5y5Eizbajnzp3LhAkT%0A6N27d4Pxxu2pAQ4ePMiUKVN48MEH68Zq20lPmjSp1TWLiDQllAvK3H0mMLPZBdtIZWUlS5cu5fjx%0A4xw/fhyAJUuWkJOTQ3p6eqvXO2PGDIYNG0ZaWhqPPvroKZfbvXs3L774Yl2vofr27NlzUjuHWbNm%0AMXHiRCZMmFA3pnbSIpIokWg6d+DAgZP6+6SkpHDgwIG41tvSNtRvvvkm7733HhdffDHZ2dl8+umn%0AXHzxxUDD9tS1Nm/ezDXXXNNgTO2kRSRRItFiIiMjg5qamgZjNTU1ZGRkxLXe2jbUO3fu5N5772Xu%0A3LlNLjdu3LgGdxvr1q1bXUvq+u2pa/30pz+tC4paaictIokSiRlBeno6hYWFpKamcs4555Camkph%0AYWFcu4XOtA31qVxzzTW89tprDcaeffZZ9uzZ02BM7aRFJFEiMSOA4GbvOTk5HDhwgIyMjLhCAGDq%0A1KlMnToVCHYzrV+/HoBRo0Y1+9lPPvmk7nnfvn3p1asX7777bl2DuCeeeKLB8nv37uXIkSPk5+fH%0AVbOISFMiMSOolZ6eTp8+feIOgbY2e/bsk2YA9ZWVlfGrX/0qiRWJSJREZkbQnvXv35/+/fuf8v1h%0Aw4YlsRoRiZqzekZwNtxd7Wyj71Qkes7aIEhLS2Pfvn364WpD7s6+ffvqrmgWkWg4a3cNZWVlUV5e%0AjjqTtq20tDSysrLCLkNEkuisDYLOnTuTk5MTdhkiIme9s3bXkIiItA0FgYhIxCkIREQiTkEgIhJx%0ACgIRkYhTEIiIRJyCQEQk4hQEIiIRpyAQEYk4BYGISMQpCEREIk5BICIScQoCEZGIUxCIiEScgkBE%0AJOIUBCIiEacgEBGJOAWBiEjEKQhERCJOQSAiEnEKAhGRiFMQiIhEnIJApKOqqoB9xcGjyGmkhl2A%0AiCRA6SJYXwSdusCJY/DV+ZA9OeyqpJ3SjECko6mqCEKg5ghUHwwe1xdpZiCn1OIZgZmlABfU/4y7%0AlyWiKBGJQ2VpMBOoOfLZWKfOwXhaZlhVSTvWoiAwsx8BM4G9wInYsAOXtWajZpYBPAHkxdZzi7uv%0Aa826RKSR9Oxgd1B9J6qDcZEmtHRGcCfQ3933tdF2HwGWuftEM+sCfK6N1isiaZnBMYH1RcFM4ER1%0A8FqzATmFlgbBB8DBttigmX0eGAncDODux4Bjp/uMiJyh7Mlw4Zhgd1B6tkJATuu0QWBmP449fR9Y%0AZWb/Fzha+767/7oV28wBKoD/MLNBwAbgTnevbMW6RORU0jIVANIizZ011D32XxmwHOhSb6xbK7eZ%0ACgwB/t3dBwOVwH2NFzKzH5hZiZmVVFTobAcRkUQ57YzA3WcBmNl33P3F+u+Z2Xdauc1yoNzd18de%0A/54mgsDd5wHzAAoKCryV2xIRkWa09DqCn7RwrFnu/iHwgZn1jw2NBra3Zl0iIhK/5o4RXANcC/Qx%0As0frvdUDOB7Hdn8ELIydMfQ+8N/iWJeIiMShubOGdgMlwASCg7q1DgN3tXaj7r4JKGjt50VEpO00%0Ad4zgLeAtM3vW3auTVJOIiCRRS68j2GhmjQ/YHiSYLfyvNrzQTEREkqylQfAnoAZ4Nvb6uwRXA38I%0ALAC+1eaViYhIUrQ0CMa4+5B6r7eY2UZ3H2JmUxJRmIiIJEdLTx9NMbOv1L4ws2FASuxlPGcPiYhI%0AyFo6I5gGPGlm3QADDgHTzCwd+N+JKk5ERBKvRUHg7sVAfqxhHO5evwHdC4koTEREkqOl9yM4B/g2%0AkA2kmhkA7v5vCatMRESSoqW7hpYQnC66gXrdR0VE5OzX0iDIcverE1qJiIiEoqVnDb1uZvkJrURE%0ARELR0hnBvwA3m9lOgl1DBri7t+qexSIi0n60NAiuSWgVIiISmhbtGnL3fwAXAaNizz9t6WdFRKR9%0Aa9GPuZnNBO7ls5vRdAZ+l6iiREQkeVr6r/rrCe5JUAng7rsJ7lssIiJnuZYGwTF3d8ABYq0lRESk%0AA2hpELxgZv8HyDCzW4FXgd8mriwREUmWlvYammNmYwmazfUHZrj78oRWJiIiSdHS00eJ/fDrx19E%0ApIM5bRCY2WFixwUav0VwQVmPhFQlIiJJ09zN63VmkIhIB6eLwkREIk5BICIScQoCEZGIUxCIiESc%0AgkBEJOIUBCIiEacgEBGJOAWBiEjEKQhERCJOQSAiEnEKAhGRiFMQiIhEnIJARCTiFAQiIhEXWhCY%0AWYqZvWlmL4dVg4iIhDsjuBN4O8Tti4gIIQWBmWUB44Anwti+iIh8JqwZwcPAPcCJkLYvIiIxSQ8C%0AMxsPfOTuG5pZ7gdmVmJmJRUVFUmqTkQkesKYEQwHJphZKfAcMMrMftd4IXef5+4F7l6QmZmZ7BpF%0ARCIj6UHg7j9x9yx3zwa+C6x09ynJrkNERAK6jkBEJOJSw9y4u68CVoVZg4hI1GlGICIScQoCEZGI%0AUxCIiEScgkBEJOIUBCIiEacgEBGJOAWBiEjEKQhERCJOQSAiEnEKAhGRiFMQiIhEnIJARCTiFAQi%0AIhGnIBARiTgFgYhIxCkIREQiTkEgIhJxCgIRkYhTEIiIRJyCQEQk4hQEIiIRF4kgqKiA4uLgUURE%0AGurwQbBoEfTtC2PHBo+LFoVdkYhI+9Khg6CiAoqK4MgROHgweCwq0sxARKS+Dh0EpaXQpUvDsc6d%0Ag3EREQl06CDIzoZjxxqOVVcH4yIiEujQQZCZCfPnQ9eu0KNH8Dh/fjAuIiKB1LALSLTJk2HMmGB3%0AUHa2QkBEpLEOHwQQ/PgrAEREmtahdw2JiEjzFAQiIhGnIBARiTgFgYhIxCkIREQiTkEgIhJxCgIR%0AkYhTEIiIRFzSg8DMLjKzv5jZdjPbZmZ3JrsGERH5TBhXFh8H7nb3jWbWHdhgZsvdfXsItYiIRF7S%0AZwTuvsfdN8aeHwbeBvokuw4REQmEeozAzLKBwcD6Jt77gZmVmFlJhe4kIyKSMKEFgZl1A/4T+Fd3%0AP9T4fXef5+4F7l6QqY5xIiIJE0oQmFlnghBY6O6Lw6hBREQCYZw1ZMB84G13/3Wyty8iIg2FMSMY%0ADnwPGGVmm2L/XRtCHSIiQginj7r7GsCSvV0REWmariwWEYk4BYGISMQpCEREIk5BICIScQoCEZGI%0AUxCIiEScgkBEJOIiEQSVlZXs2rWLysrKsEsREWl3wrgfQVJt2bKFpUuXkpKSQk1NDYWFheTl5YVd%0AlohIu9GhZwSVlZUsXbqU48ePc/ToUY4fP86SJUs0MxARqadDB8GBAwdISUlpMJaSksKBAwdCqkhE%0ApP3p0EGQkZFBTU1Ng7GamhoyMjJCqkhEpP3p0EGQnp5OYWEhqampnHPOOaSmplJYWEh6enrYpYmI%0AtBsd/mBxXl4eOTk5HDhwgIyMDIWAiEgjHT4IIJgZKABERJrWoXcNiYhI8xQEIiIRpyAQEYk4BYGI%0ASMQpCEREIk5BICIScQoCEZGIM3cPu4ZmmVkF8I+w62iB84B/hl1EO6bv5/T0/ZyavpvTO9X309fd%0AM5v78FkRBGcLMytx94Kw62iv9P2cnr6fU9N3c3rxfj/aNSQiEnEKAhGRiFMQtK15YRfQzun7OT19%0AP6em7+b04vp+dIxARCTiNCMQEYk4BUEbMLOLzOwvZrbdzLaZ2Z1h19TemFmKmb1pZi+HXUt7Y2YZ%0AZvZ7M9thZm+b2eVh19SemNldsf+vtprZIjNLC7umMJnZk2b2kZltrTd2rpktN7N3Y489z2SdCoK2%0AcRy4290vBb4G/A8zuzTkmtqbO4G3wy6inXoEWObuA4BB6HuqY2Z9gDuAAnfPA1KA74ZbVegWAFc3%0AGrsPWOHuucCK2OsWUxC0AXff4+4bY88PE/yP3CfcqtoPM8sCxgFPhF1Le2NmnwdGAvMB3P2Yux8I%0At6p2JxXoamapwOeA3SHXEyp3fw3Y32i4EHgq9vwp4LozWaeCoI2ZWTYwGFgfbiXtysPAPcCJsAtp%0Ah3KACuA/YrvOnjAz3U4vxt13AXOAMmAPcNDdXwm3qnbpAnffE3v+IXDBmXxYQdCGzKwb8J/Av7r7%0AobDraQ/MbDzwkbtvCLuWdioVGAL8u7sPBio5w2l9Rxbb111IEJhfANLNbEq4VbVvHpwKekangyoI%0A2oiZdSYIgYXuvjjsetqR4cAEMysFngNGmdnvwi2pXSkHyt29dgb5e4JgkMAYYKe7V7h7NbAY+HrI%0ANbVHe82sN0Ds8aMz+bCCoA2YmRHs433b3X8ddj3tibv/xN2z3D2b4CDfSnfXv+hi3P1D4AMz6x8b%0AGg1sD7Gk9qYM+JqZfS72/9lodDC9KUuB78eefx9YciYfVhC0jeHA9wj+tbsp9t+1YRclZ40fAQvN%0AbDPwZeDBkOtpN2Izpd8DG4EtBL9Zkb7K2MwWAeuA/mZWbmZFwGxgrJm9SzCLmn1G69SVxSIi0aYZ%0AgYhIxCkIREQiTkEgIhJxCgIRkYhTEIiIRJyCQKQVYq0gTttY0MwWmNnEJsazzezGxFUncmYUBCKt%0A4O7T3L21F35lAwoCaTcUBBJpZjbdzO6IPf+Nma2MPR9lZgvN7Jtmts7MNprZi7F+UpjZKjMriD0v%0AMrN3zOwNM/utmc2tt4mRZva6mb1fb3YwGxgRu/DwriT+uSJNUhBI1K0GRsSeFwDdYn2jRgCbgfuB%0AMe4+BCgBflz/w2b2BeABgvtQDAcGNFp/b+BfgPF8drXnfcBqd/+yu/+mzf8ikTOUGnYBIiHbAAw1%0Asx7AUYJWBgUEQbAUuBRYG7S5oQvBpf31fQX4q7vvBzCzF4F+9d7/L3c/AWw3szNqDSySLAoCiTR3%0ArzazncDNwOsEs4ArgYuBncByd58cxyaO1ntucaxHJGG0a0gk2D30P4HXYs//O/Am8P+A4WZ2MYCZ%0ApZtZv0afLQauMLOesTtofbsF2zsMdG+r4kXipSAQCX78ewPr3H0vUEWwD7+CYKawKNYZdB2NjgHE%0A7qD1IPAGsBYoBQ42s73NQI2ZvaWDxdIeqPuoSJzMrJu7fxKbEfwBeNLd/xB2XSItpRmBSPx+bmab%0AgK0ExxX+K+R6RM6IZgQiIhGnGYGISMQpCEREIk5BICIScQoCEZGIUxCIiEScgkBEJOL+P3vS2BJ7%0AeG9SAAAAAElFTkSuQmCC)

#### 유클리드 거리
유클리드 거리의 수식은 다음과 같다.

$$\sqrt{\sum^n_{i=1} (x_i - y_i)^2}$$

코드와 계산 결과는 아래와 같다.

```python
def euclidean_distance(x, y):   
    return np.sqrt(np.sum((x - y) ** 2))
```

```python
x0 = X[0][:-1]
x1 = X[1][:-1]
x4 = X[4][:-1]
x14 = X[14][:-1]
print(" x0:", x0, "\n x1:", x1, "\n x4:", x4, "\nx14:", x14)

print(" x14 and x0:", euclidean_distance(x14, x0), "\n",
      "x14 and x1:", euclidean_distance(x14, x1), "\n",
      "x14 and x4:", euclidean_distance(x14, x4))
--

 x0: [6.6 6.2] 
 x1: [9.7 9.9] 
 x4: [1.3 2.7] 
x14: [1.3 1.3]

 x14 and x0: 7.218032973047436 
 x14 and x1: 12.021647141718974 
 x14 and x4: 1.4000000000000001
```

유클리드 거리에 따르면 4번 인스턴스와 가장 가까우며, 따라서 k-NN을 적용했을때 young 클래스로 추측할 수 있다. 우리의 직관과 일치하는 나쁘지 않은 결과다.

#### 코사인 유사도
이제 코사인 유사도를 적용해보자. 수식은 아래와 같다.

$$\frac{x \bullet y}{ \sqrt{x \bullet x} \sqrt{y \bullet y}}$$

마찬가지로 코드 구현과 값을 출력해보도록 한다.

```python
def cosine_similarity(x, y):
    return np.dot(x, y) / (np.sqrt(np.dot(x, x)) * np.sqrt(np.dot(y, y)))

print(" x14 and x0:", cosine_similarity(x14, x0), "\n",
      "x14 and x1:", cosine_similarity(x14, x1), "\n",
      "x14 and x4:", cosine_similarity(x14, x4))
--
 x14 and x0: 0.9995120760870786 
 x14 and x1: 0.9999479424242859 
 x14 and x4: 0.9438583563660174
```

코사인 유사도에 따르면 14번은 이번에는 1번과 가장 가까운 것으로 나온다. 1번은 adult 클래스로 우리가 기대하는 바와는 다소 다른 결과이며, 뿐만 아니라 유클리드 거리에서 가장 가까웠던 4번 인스턴스는 오히려 가장 먼 것으로 나온다.

### 무슨 일이 일어 났을까
유클리드 거리($$d$$)와 코사인 유사도($$\theta$$)를 시각적으로 표현하면 아래와 같다.

![](http://semanticvoid.com/images/cosine_similarity.png)

유클리드 거리는 줄자로 거리를 측정하는 것과 유사하며, 코사인 유사도는 무게나 크기는 전혀 고려하지 않고 벡터 사이의 각도만으로 측정하는 것과 유사하다.

즉, 14번과 4번은 줄자로 쟀을때는(유클리드 거리) 가장 가깝게 나오지만 각도를 쟀을때는(코사인 유사도) 가장 낮은 값이 되었던 것이다.

### 언제 코사인 유사도를 사용하는가
그렇다면 코사인 유사도는 언제 사용할까? 

일반적으로 코사인 유사도는 벡터의 크기가 중요하지 않을때 거리를 측정하기 위한 메트릭으로 사용된다. 예를 들어 단어의 포함 여부로 문서의 유사 여부를 판단한다고 할때 'science'라는 단어가 2번 보다 1번 문서에 더 많이 포함되어 있다면 1번 문서가 과학 문서라고 추측할 수 있을 것이다. 그러나, 만약 1번 문서가 2번 문서 보다 훨씬 더 길다면 공정하지 않은 비교가 된다. 이때 코사인 유사도는 이 문제를 바로 잡아줄 수 있다.

즉, 길이를 정규화해 비교하는 것과 유사하다고 할 수 있으며 이 때문에 텍스트 데이터를 처리하는 메트릭으로 주로 사용된다. 주로 데이터 마이닝이나 정보 검색<sup>information retrieval</sup>에서 즐겨 사용된다.

#### 코사인 유사도 예제
그렇다면 실제 예제를 살펴보면서 코사인 유사도가 어떤 역할을 하는지 살펴보자.

```python
import wikipedia

q1 = wikipedia.page('Machine Learning')
q2 = wikipedia.page('Artifical Intelligence')
q3 = wikipedia.page('Soccer')
q4 = wikipedia.page('Tennis')

```

위키피디어에서 4개의 문서를 가져온다.

```python
q1.content[:100]
--
'Machine learning is a field of computer science that often uses statistical techniques to give compu'

q1.content.split()[:10]
--
['Machine',
 'learning',
 'is',
 'a',
 'field',
 'of',
 'computer',
 'science',
 'that',
 'often']

print("ML \t", len(q1.content.split()), "\n"
      "AI \t", len(q2.content.split()), "\n"
      "soccer \t", len(q3.content.split()), "\n"
      "tennis \t", len(q4.content.split()))
--
ML 	 4048 
AI 	 13742 
soccer 	 6470 
tennis 	 9736
```

변수에는 각각의 본문이 들어가며, 문서의 길이는 모두 다르다.

```python
from sklearn.feature_extraction.text import CountVectorizer

cv = CountVectorizer()
X = np.array(cv.fit_transform([q1.content, q2.content, q3.content, q4.content]).todense())
```
이를 k-hot vector로 인코딩 한다.

```python
X[0].shape
--
(5484,)

X[0][:20]
--
array([0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0],
      dtype=int64)
```

X는 전체 단어수 만큼의 배열이며, 각각의 값은 해당 단어의 출현 빈도를 나타낸다.

이제 이 문서들간의 유사도를 유클리드 거리로 나타내보자.

```python
print("ML - AI \t", euclidean_distance(X[0], X[1]), "\n"
      "ML - soccer \t", euclidean_distance(X[0], X[2]), "\n"
      "ML - tennis \t", euclidean_distance(X[0], X[3]))
--
ML - AI 	 846.53411035823 
ML - soccer 	 479.75827246645787 
ML - tennis 	 789.7069076562519
```

ML 문서는 축구 문서와 가장 가깝고 AI 문서와는 가장 먼 것으로 나타난다. 일반적으로 우리가 생각하는 결과와 다르다. 이는 문서의 길이가 다르기 때문인데, 이번에는 코사인 유사도로 한 번 비교해보자.

```python
print("ML - AI \t", cosine_similarity(X[0], X[1]), "\n"
      "ML - soccer \t", cosine_similarity(X[0], X[2]), "\n"
      "ML - tennis \t", cosine_similarity(X[0], X[3]))
--
ML - AI 	 0.8887965704386804 
ML - soccer 	 0.7839297821715802 
ML - tennis 	 0.7935675914311315
```

AI 문서가 가장 높은 값, 축구 문서가 가장 낮은 값으로 앞서 유클리드 거리와는 정반대의 결과이며 이는 우리의 직관과 일치하는 결과를 보여준다. 그렇다면 이번에는 문서의 길이를 정규화 해서 유클리드 거리로 다시 한 번 비교해보도록 한다.

```python
def l2_normalize(v):
    norm = np.sqrt(np.sum(np.square(v)))
    return v / norm

print("ML - AI \t", 1 - euclidean_distance(l2_normalize(X[0]), l2_normalize(X[1])), "\n"
      "ML - soccer \t", 1 - euclidean_distance(l2_normalize(X[0]), l2_normalize(X[2])), "\n"
      "ML - tennis \t", 1 - euclidean_distance(l2_normalize(X[0]), l2_normalize(X[3])))
--
ML - AI 	 0.5283996828641448 
ML - soccer 	 0.3426261066509869 
ML - tennis 	 0.3574544240773757
```

코사인 유사도와 값은 다르지만 패턴은 일치한다. AI 문서가 가장 높은 값, 축구 문서가 가장 낮은 값으로 길이를 정규화해 유클리드 거리로 비교한 결과는 코사인 유사도와 거의 유사한 패턴을 보인다.

#### 트위터 분류
또 다른 예제를 살펴보자. 오픈AI의 트윗이다.

```python
ml_tweet = "New research release: overcoming many of Reinforcement Learning's limitations with Evolution Strategies."
x = np.array(cv.transform([ml_tweet]).todense())[0]
```

당연히 ML 또는 AI와 유사한 결과가 나와야 할 것 같다.

```python
print("tweet - ML \t", euclidean_distance(x, X[0]), "\n"
      "tweet - AI \t", euclidean_distance(x, X[1]), "\n"
      "tweet - soccer \t", euclidean_distance(x, X[2]), "\n"
      "tweet - tennis \t", euclidean_distance(x, X[3]))
--
tweet - ML 	 373.09114167988497 
tweet - AI 	 1160.7269274036853 
tweet - soccer 	 712.600168397398 
tweet - tennis 	 1052.5796881946753
```

그러나 유클리드 거리로 계산한 결과는 축구 문서가 AI 문서보다 오히려 더 가까운 것으로 나온다. 이제 코사인 유사도 결과를 살펴보자.

```python
print("tweet - ML \t", cosine_similarity(x, X[0]), "\n"
      "tweet - AI \t", cosine_similarity(x, X[1]), "\n"
      "tweet - soccer \t", cosine_similarity(x, X[2]), "\n"
      "tweet - tennis \t", cosine_similarity(x, X[3]))
--
tweet - ML 	 0.2613347291026786 
tweet - AI 	 0.19333084671126158 
tweet - soccer 	 0.1197543563241326 
tweet - tennis 	 0.11622680287651725
```

AI 문서가 축구 문서 보다 훨씬 더 유사한 값으로 나온다. 이번에는 길이를 정규화해 유클리드 거리로 비교한 결과는 어떨까.

```python
print("tweet - ML \t", 1 - euclidean_distance(l2_normalize(x), l2_normalize(X[0])), "\n"
      "tweet - AI \t", 1 - euclidean_distance(l2_normalize(x), l2_normalize(X[1])), "\n"
      "tweet - soccer \t", 1 - euclidean_distance(l2_normalize(x), l2_normalize(X[2])), "\n"
      "tweet - tennis \t", 1 - euclidean_distance(l2_normalize(x), l2_normalize(X[3])))
--
tweet - ML 	 -0.2154548703241279 
tweet - AI 	 -0.2701725499228351 
tweet - soccer 	 -0.32683506410998 
tweet - tennis 	 -0.3294910282687
```

값이 작아서 음수로 나타나긴 했지만 마찬가지로 AI 문서가 축구 문서 보다 더 높은 값을 잘 나타내는 것을 확인할 수 있다.

```python
so_tweet = "#LegendsDownUnder The Reds are out for the warm up at the @nibStadium. Not long now until kick-off in Perth."
x2 = np.array(cv.transform([so_tweet]).todense())[0]
```

이번에는 맨체스터 유나이티드의 트윗을 살펴보자.

```python
print("tweet - ML \t", euclidean_distance(x2, X[0]), "\n"
      "tweet - AI \t", euclidean_distance(x2, X[1]), "\n"
      "tweet - soccer \t", euclidean_distance(x2, X[2]), "\n"
      "tweet - tennis \t", euclidean_distance(x2, X[3]))
--
tweet - ML 	 371.8669116767449 
tweet - AI 	 1159.1397672412072 
tweet - soccer 	 710.1035135809426 
tweet - tennis 	 1050.1485609188826
```

유클리드 거리는 ML 문서가 축구 문서보다 더 가깝다고 잘못 얘기한다.

```python
print("tweet - ML \t", cosine_similarity(x2, X[0]), "\n"
      "tweet - AI \t", cosine_similarity(x2, X[1]), "\n"
      "tweet - soccer \t", cosine_similarity(x2, X[2]), "\n"
      "tweet - tennis \t", cosine_similarity(x2, X[3]))
--
tweet - ML 	 0.4396242958582417 
tweet - AI 	 0.46942065152331963 
tweet - soccer 	 0.6136116162795926 
tweet - tennis 	 0.5971160690477066
```

그러나 코사인 유사도는 축구 문서가 훨씬 더 유사하다.

```python
print("tweet - ML \t", 1 - euclidean_distance(l2_normalize(x2), l2_normalize(X[0])), "\n"
      "tweet - AI \t", 1 - euclidean_distance(l2_normalize(x2), l2_normalize(X[1])), "\n"
      "tweet - soccer \t", 1 - euclidean_distance(l2_normalize(x2), l2_normalize(X[2])), "\n"
      "tweet - tennis \t", 1 - euclidean_distance(l2_normalize(x2), l2_normalize(X[3])))
--
tweet - ML 	 -0.0586554719470902 
tweet - AI 	 -0.030125573390623384 
tweet - soccer 	 0.12092277504145588 
tweet - tennis 	 0.10235426703816686
```

길이를 정규화한 유클리드 거리 계산 결과도 축구 문서가 가장 높은 동일한 패턴을 보이며 이는 우리의 직관과 일치하는 만족스런 결과를 보여준다.

## 참고
전체 코드를 포함한 주피터 노트북으로 직접 계산해본 결과는 [euclidean-v-cosine.ipynb](https://nbviewer.jupyter.org/github/likejazz/jupyter-notebooks/blob/master/data-science/euclidean-v-cosine.ipynb)에서 확인할 수 있으며, 원문은 [Euclidean vs. Cosine Distance](https://cmry.github.io/notes/euclidean-v-cosine)에서 볼 수 있다.