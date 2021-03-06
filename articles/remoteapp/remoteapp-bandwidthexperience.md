<properties 
    pageTitle="Azure RemoteApp - ネットワーク帯域幅とエクスペリエンスの質はどのような関係にあるのか | Microsoft Azure"
	description="Azure RemoteApp でネットワーク帯域幅がユーザー エクスペリエンスの質に与える影響について説明します。"
	services="remoteapp"
	documentationCenter="" 
	authors="lizap" 
	manager="mbaldwin" />

<tags 
    ms.service="remoteapp" 
    ms.workload="compute" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="03/31/2016" 
    ms.author="elizapo" />

# Azure RemoteApp - ネットワーク帯域幅とエクスペリエンスの質はどのような関係にあるのか

Azure RemoteApp に必要な[全体的なネットワーク帯域幅](remoteapp-bandwidth.md)を調べる場合は、以下の点を念頭に置いてください。これらはすべて動的なシステムの一部であり、全体的なユーザー エクスペリエンスに影響します。

- **利用可能なネットワーク帯域幅と現在のネットワークの状態** - ある時点における同一ネットワーク上の一連のパラメーター (損失、遅延、ジッター) はアプリケーションのストリーミングに影響を与える可能性がありますが、これは全体的なユーザー エクスペリエンスの低下を意味します。ネットワークで利用可能な帯域幅は、輻輳の状態、ランダムな損失、遅延と相関関係にあります。これは、それらのパラメーターが輻輳制御メカニズムに影響を与えた結果、競合を避けて伝送速度が制御されるためです。たとえば、帯域幅が 1000 MB のネットワーク上でも、損失の多いネットワークや遅延の大きいネットワークではユーザー エクスペリエンスの質が低下します。損失と遅延は同じネットワーク上のユーザー数とそれらのユーザーが行う操作に応じて変わります (ビデオの視聴、サイズの大きいファイルのダウンロードやアップロード、印刷など)。
- **使用シナリオ** - ユーザー エクスペリエンスは、同じネットワーク上でユーザーが個人またはグループでどのような操作を行うかに応じて変わります。たとえば、1 枚のスライドを表示する場合は 1 つのフレームの更新だけで済みますが、テキスト ドキュメントをスクロールしながら流し読みする場合は、更新が必要な 1 秒あたりのフレーム数が増加します。このシナリオではサーバーとのやり取りが行われた結果、より多くのネットワーク帯域幅が消費されます。また、極端な例としては、複数のユーザーによる (4K 解像度などの) 高解像度ビデオの視聴、HD 電話会議の開催、3D ビデオ ゲームの再生、CAD システムでの作業などがあります。高帯域幅のネットワークでも、これらの要因が実質的な操作性の低下につながる場合があります。
- **画面の解像度や画面の数** - 大きい画面で全画面を更新する場合は、小さい画面よりも必要なネットワーク帯域幅が増加します。基になるテクノロジの優れた性能により、画面内の更新された領域のみがエンコードおよび転送されますが、以前は画面全体を更新する必要がありました。高解像度の画面 (4K 解像度など) の場合、更新に必要なネットワーク帯域幅は低解像度の画面 (1024x768px) よりも増加します。リダイレクトに複数の画面を使用する場合にも同じことが言えます。画面の数に応じて帯域幅を増やす必要があります。
- **クリップボードとデバイス リダイレクト** - これはあまり明白な問題ではありませんが、多くの場合、ユーザーがクリップボードに大きなデータを格納すると、リモート デスクトップ クライアントからサーバーへのデータ転送に一定の時間がかかります。ダウンストリームのエクスペリエンスは、クリップボードの内容のアップストリーム送信により影響を受ける場合があります。デバイス リダイレクトでも同様です。たとえば、スキャナーや Web カメラが生成された大量のデータをアップストリームのサーバーに送信する必要がある場合、プリンターで大きいドキュメントを受信する必要がある場合、またはクラウドで実行中のアプリがローカル ストレージに大きいファイルをコピーする必要がある場合は、デバイス リダイレクトに必要なデータによってネットワーク帯域幅のニーズが増加するため、ユーザーがフレームの欠落や一時的に "固まった" ビデオに気づく場合があります。 

ネットワーク帯域幅のニーズを評価する場合は、必ずシステムの一部である上記の要素を考慮してください。

[ネットワーク帯域幅に関するメインの記事](remoteapp-bandwidth.md)に戻るか、[ネットワーク帯域幅](remoteapp-bandwidthtests.md)のテストに進んでください。

## 詳細情報
- [Azure RemoteApp で使用されるネットワーク帯域幅を推定する](remoteapp-bandwidth.md)

- [Azure RemoteApp - testing your network bandwidth usage with some common scenarios (Azure RemoteApp - 一般的なシナリオでのネットワークの使用帯域幅をテストする)](remoteapp-bandwidthtests.md)

- [Azure RemoteApp network bandwidth - general guidelines (if you can't test your own) (Azure RemoteApp のネットワーク帯域幅に関する一般的なガイドライン (自分でテストできない場合))](remoteapp-bandwidthguidelines.md)

<!---HONumber=AcomDC_0406_2016-->