---
title: "[Spready] 착시 연결 기믹을 구현한 방법"

categories:
  - Development Log
tags:
  - [Spready]

toc: true
toc_sticky: true

date: 2024-10-23
last_modified_at: 2024-10-27
---

<!-- <img src="https://seoh136199.github.io/assets/img/Main_Capsule_616_358.png" style="display: block; margin-left: 0;"> -->
<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/img/Main_Capsule_616_358.png" style="width: 100%; display: block; border-radius: 13px;">
</figure>

[• Spready 스팀 페이지 •](https://store.steampowered.com/app/2944240/Spready/?l=korean "spready_steam")<br>
[• Spready 플레이스토어 •](https://play.google.com/store/apps/details?id=com.SpitzerGames.Spready&hl=ko "spready_ps")<br>
[• Spready Demo 플레이스토어 •](https://play.google.com/store/apps/details?id=com.SpitzerGames.SpreadyDemo&hl=ko "spready_demo_ps")<br><br>

---

# 게임 소개

Spready에 대해 간단한 소개부터 하자면, 4개 챕터로 이루어진 <b>색칠하는 3D 퍼즐 게임</b>이다.

<!-- <img src="https://seoh136199.github.io/assets/gif/스팀움짤1.gif" style="display: block; margin-left: 0;"> -->
<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center; ">
    <img src="https://seoh136199.github.io/assets/gif/스팀움짤1.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
각 스테이지에는 캔버스가 배치되어 있다. 플레이어는 크레파스와 지우개를 사용해 모든 캔버스를 색칠하면 된다.<br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/스팀움짤2.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
거기에 휘어진 캔버스도 등장한다.  <del>크레파스가 딱딱할 줄 알았겠지만 사실 아니다 </del><br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/스팀움짤3.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
2챕터부터는 캔버스를 회전시킬 수 있는 바퀴도 보이기 시작한다.<br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/스팀움짤4.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
3챕터부터는 카메라 각도를 잘 맞추면 크레파스나 지우개를 복제할 수 있다.<br><br>

그럼 4챕터엔 뭐가 나올까?<br>
이 글에서 설명하려고 하는 <b>착시 연결 기믹</b>이다.

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/스팀움짤5.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>

두 캔버스가 실제로 닿아있지 않아도, 카메라 각도를 조절해서 닿아있는 것처럼 보이게 만들면 크레파스가 그 사이를 지나가며 색칠할 수 있게 해주는 기믹이다.<br><br>

별거 아닌 것처럼 보일 수 있지만, 착시 효과를 구현하면서도 이상하게 보이는 부분이 없도록 하기 위해 많은 시행착오를 겪었다. 이 글에서는 실패했던 방법과, 제대로 구현하는 데에 성공한 방법을 소개하려고 한다.<br><br>

---

# 실패한 방법

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/초기상태.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
이 기믹을 구현할 때 특별한 처리를 하지 않고 크레파스를 이동시키면 이렇게 보인다.
먼 쪽의 캔버스를 지나는 동안은 가까운 쪽의 캔버스에 가려져있다가, 어느 순간 가까운 쪽의 캔버스 위로 올라와있게 된다.<br>
하지만 두 캔버스가 평평하게 맞닿아있는 것처럼 보이도록 착시를 일으켜야 하기 때문에, 크레파스가 캔버스에 가려지면 안 된다.<br>

여기서 떠올린 아이디어는 <b>레이어를 분리</b>하는 것이다.<br><br><br>

## 레이어 분리 방법

<div style="display: flex; justify-content: flex-start; align-items: flex-start; width: 100%;">
    <figure style="margin: 0; padding: 0; width: 35%; text-align: center;">
        <img src="https://seoh136199.github.io/assets/img/2레이어-1.png" style="width: 100%; display: block; border-radius: 8px;">
        <figcaption style="font-size: 12px; padding: 4">나머지 레이어</figcaption>
    </figure>
    <figure style="margin: 0; margin-left: 10px; padding: 0; width: 35%; text-align: center;">
        <img src="https://seoh136199.github.io/assets/img/2레이어-2.png" style="width: 100%; display: block; border-radius: 8px;">
        <figcaption style="font-size: 12px; padding: 4">이동 중인 크레파스 레이어</figcaption>
    </figure>
</div><br>

이동 중인 크레파스 레이어와 나머지 레이어를 분리해서, 이동 중인 크레파스 레이어가 캔버스보다 무조건 앞에 보이게 하면 앞서 설명한 문제가 해결된다.<br>
(레이어 이미지에서 검은색인 부분은 실제로는 투명해서 뒷 레이어에 그려진 게 투과되어 보이는 부분이다)<br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/실패1.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
크레파스가 어느 순간 올라온 것처럼 보이는 문제가 해결된 걸 확인할 수 있다.<br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/실패2.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
하지만 가까운 오브젝트가 크레파스를 가리는 경우가 또다른 문제임을 알게 되었다.<br>
이 경우, 주황색 크레파스와 위쪽 캔버스는 크레파스를 가리는 위치에 있으므로 크레파스보다 앞에 보여야 한다. 그러나 크레파스 레이어 때문에 이들이 가려지는 문제가 발생한다.<br>
이동 중인 크레파스가 모든 오브젝트보다 항상 앞에 보이면 안 되며, 카메라에 더 가까운 오브젝트보다는 뒤에 보여야 한다.<br><br><br>

## 레이어 3단 분리 방법

<div style="display: flex; justify-content: flex-start; align-items: flex-start; width: 100%;">
    <figure style="margin: 0; padding: 0; width: 26%; text-align: center;">
        <img src="https://seoh136199.github.io/assets/img/3레이어-1.png" style="width: 100%; display: block; border-radius: 8px;">
        <figcaption style="font-size: 12px; padding: 4">먼 오브젝트 레이어</figcaption>
    </figure>
    <figure style="margin: 0; margin-left: 10px; padding: 0; width: 26%; text-align: center;">
        <img src="https://seoh136199.github.io/assets/img/3레이어-2.png" style="width: 100%; display: block; border-radius: 8px;">
        <figcaption style="font-size: 12px; padding: 4">이동 중인 크레파스 레이어</figcaption>
    </figure>
    <figure style="margin: 0; margin-left: 10px; padding: 0; width: 26%; text-align: center;">
        <img src="https://seoh136199.github.io/assets/img/3레이어-3.png" style="width: 100%; display: block; border-radius: 8px;">
        <figcaption style="font-size: 12px; padding: 4">가까운 오브젝트 레이어</figcaption>
    </figure>
</div><br>
그래서 <b>레이어를 추가</b>했다.<br>
크레파스를 가릴만한 위치에 있는 오브젝트를 분류해서 가까운 오브젝트 레이어에 넣어준 뒤, 이동 중인 크레파스 레이어보다 항상 앞에 보이게 해줬다.<br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/실패3.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
잘 작동한다! 앞에서 말한 모든 문제들이 해결됐다.<br><br><br>

## 그러나...

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/실패사례1.gif" style="width: 100%; display: block; border-radius: 8px;">
    <figcaption style="font-size: 12px; padding: 4">문제 사례 1</figcaption>
</figure>
<figure style="margin: 0; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/실패사례2.gif" style="width: 100%; display: block; border-radius: 8px;">
    <figcaption style="font-size: 12px; padding: 4">문제 사례 2</figcaption>
</figure>
하지만 착각이었다. 위와 같은 문제 사례가 계속 발견되었고, 레이어 분리만으로는 해결할 수 없거나 완전히 해결하기 매우 어려운 문제임을 깨달았다.<br><br>

---

# 성공한 방법

두 캔버스가 실제로는 떨어져 있지만 연결되어 있는 것처럼 크레파스가 지나가야 한다. 이때 가까운 캔버스에서는 크레파스도 가까이에 있고, 먼 캔버스에서는 크레파스도 멀리 있게 하면 앞서 발견된 문제들이 해결된다.<br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/동강시연.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
그래서 레이어를 분리하지 않고, 크레파스 자체를 분리하기로 했다.<br><br>

<!-- <img src="https://seoh136199.github.io/assets/gif/동강이동시연.gif" style="display: block; margin-left: 0;"> -->
<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 90%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/동강이동시연.gif" style="width: 100%; display: block; border-radius: 8px;">
</figure>
카메라 각도만 유지된다면 크레파스를 동강내도 한 개의 크레파스처럼 보인다. 착시 연결 상태에서 크레파스가 이동 중일 땐 카메라 시점을 바꿀 수 없기 때문에 동강난 크레파스의 단면을 보게 될 위험은 없다.<br> 
또한, 크레파스를 분리함으로써 앞서 언급한 모든 문제가 해결되었다!<br><br>

<figure style="margin: 0; margin-bottom: 10px; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/성공사례1.gif" style="width: 100%; display: block; border-radius: 8px;">
    <figcaption style="font-size: 12px; padding: 4">문제 해결 1</figcaption>
</figure>
<figure style="margin: 0; padding: 0; width: 70%; text-align: center;">
    <img src="https://seoh136199.github.io/assets/gif/성공사례2.gif" style="width: 100%; display: block; border-radius: 8px;">
    <figcaption style="font-size: 12px; padding: 4">문제 해결 2</figcaption>
</figure><br>

---
