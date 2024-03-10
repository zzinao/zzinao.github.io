---
emoji: 🔮
title: 리팩토링에 대한 고찰1 (feat.컴파운드 패턴)
date: '2024-03-10 00:00:00'
author: zzinao
tags: React refactoring
categories: 리팩토링
---

## 1. 판도라의 상자.. 아니 컴포넌트를 열다

올해 새 업무를 들어가면서, 작년에 만든 프로젝트를 새로 개편하게 되었습니다. 프로젝트 베이스는 같지만 기능이 새로 추가되고 페이지 화면 UI가 완전히 바뀌어서 기존 코드를 기반으로 새로 짜야 하는 상황이라 끝난 줄만 알았던 그 코드들을 오랜만에 꺼내보니 경악을 금치 못했습니다. (물론 그 당시에도 진행하면서 경악했지만..)

```jsx

const OrderSummaryPage = () => {
  const {data} = useGetOrderDetail();
    return(
        <div>
        <Header />
            <OrderCard
            title={data?.title}
            content={data?.content}
            departureLocation={data?.departureLocation}
            departureAddress={data?.depatrueAddress}
            deaprtureAddressDetail={data?.deaptrueAddressDetail}
            departurePhone={data?.departruePhone}
            arrivalLocation={data?.arrivalLocation}
            arrivalAddress={data?.arrvialAddress}
            arrivalAddressDetail={data?.arrivalAddressDetail}
            isMaster={data?.masterFlag}
            isBcWrited={data?.bcWrited}
            status={data?.orderStauts}
            {...}
            // (끝없는 props의 향연)
          />
          <OrderContents
            content={data?.content}
            isMaster={data?.masterFlag}
            imageList={data?.imageUrlList}
            orderType={data?.orderType}
            {...}
          />
          <OrderLocationInfo
            departureLocation={data?.departureLocation}
            departureAddress={data?.depatrueAddress}
            deaprtureAddressDetail={data?.deaptrueAddressDetail}
            departurePhone={data?.departruePhone}
            arrivalLocation={data?.arrivalLocation}
            arrivalAddress={data?.arrvialAddress}
            arrivalAddressDetail={data?.arrivalAddressDetail}
            {...}
          />

           <OrderButton
            isMaster={data?.masterFlag}
            isBcWrited={data?.bcWrited}
            status={data?.orderStauts}
            onClick={handleClick}
            {...}
          />
          <FotterButton />
      </div>
  )
}
```

위와 같이 props를 적게는 10개 많게는 20개를 넘는 컴포넌트가 넘쳐났기 때문입니다.ㅎㅎ
물론 props를 무작정 많이 받는 게 잘못은 아니라곤 생각하지만... 저렇게 되면 1)가독성이 매우 나빠지고 2)유지 보수가 힘들고 3)에러 추적이 힘듭니다. 물론 컴포넌트가 서버 api를 받아서 보여주는 거라면 spread 문법을 통해서 간결하게 보여줄 순 있겠지만 같은 값이라도 api마다 변수명이 달랐기에.. 재 생각엔 한 번의 가공 처리가 필수였습니다.

### 🥹 잠시만! 저렇게 된 이유가 있다구요..!

> 👼🏻(반년전의 나) : 억울합니다..! 처음엔 이렇게 될 줄 몰랐다구요..!

정말 몰랐을까요? 놀랍게도 반년 전의 저는 정말 몰랐습니다.. 신입이기도 했고 기획과 동시에 개발이 들어가는 프로젝트가 처음이었기 때문입니다...여러 페이지에서 쓰이는 공통 컴포넌트를 그저 만들었을 뿐인데... 스프린트가 거듭되고 기획이 고도화될수록 이미 완성형이라고 생각한 컴포넌트에 여러 기능과 조건들과 상태 값이 추가되었습니다. 그에 따라 저는 props를 받아 컴포넌트 내에서 분기 처리를 하다 보니 한 컴포넌트에서 분기 처리하는 로직이 너무나도 많아져 점점 손대기 힘들어졌습니다.

![refactoring3.png](refactoring3.jpeg)

<br/>
그 당시에도 새로 짤까.. 하는 생각을 몇백 번 했지만 개발 기간이 너무 촉박했고 새로 설계하는 것보다 조건문 하나 더 추가하는 것이 빠른 길이었기에... 일단 돌아가게만 만들자!라는 마음에서 지금의 괴물이 태어나 버린 것입니다..

<br />

### 🛠️ 문제점 파악

당연히 지금 시점에서는 저 코드는 전량 폐기입니다. 참고도 못할 수준이거든요;; 다시는 저런 과오를 저지르지 않도록... 요번 컴포넌트는 사랑만 받을 수 있도록... 리팩토링을 진행해보려고 합니다.
일단 제가 생각하기에 제일 큰 문제점들을 꼽아 보았습니다.

#### 1. 비지니스 로직과 뷰의 분리

처음에는 데이터를 뿌려 보여주기만 한 컴포넌트였지만 점점 기능이 추가되면서 컴포넌트 내에 액션들이 필요해졌습니다. 그렇다 보니 한 공간 안에 뷰와 비즈니스 로직이 같이 존재하게 되고 이는 컴포넌트의 확장을 힘들게 만들어버립니다.

#### 2. 재활용이 전혀 안돼!

위의 이유다 보니 이 UI를 기반으로 다른 컴포넌트를 만든 디자인과 기획에 대해 대응이 매우 힘들었습니다. 기존 컴포넌트는 로직 범벅이라 또 새로운 로직을 추가하기엔 난잡해지고.. 그렇다고 똑같은 UI 중복 코드가 여러개 생기는 것도 찜찜하고.. 의존성이 강하다 못해 집중되어 있으니 이리 진퇴양난 일수가 없습니다.

## 2. Compound Pattern

그리하여 이번 리팩토링에는 Compound Pattern을 적용했습니다. 예전 프로젝트의 디자인 시스템을 만들었을 때 사용해봤지만 일반 컴포넌트엔 적용을 해보지 않아 몇 번의 시행착오를 겪어 기록해 봅니다,,

<br/>
먼저 현 서비스의 이용자는 `주문자` / `배달자` 두 분류로 나뉘고 두 페이지는 거의 비슷한 ui 형태를 가지고 있습니다. `주문 상태`에 따라 ui가 바뀌는데 저번과 동일하게 기획과 동시에 개발이 들어가기 때문에 ui와 기능이 어떻게 수정되고 추가될지 몰라 확장성을 염두에 두고 개발에 들어가야 합니다.
<br/>

### 1. 첫 번째 시도

```jsx
cosnt OrderSummaryPage = () => {
const {data} = useGetOrderDetail();

  return(
    <div>
    <Header>
    <OrderSummary {...data} isOrderer={true}>
      <OrderSummary.Card />
      <OrderSummary.Contents />
      <OrderSummary.LocationInfo />
      <OrderSummary.Button />
    </OrderSummary>
    <FooterButton/>
    </div>
  )
}
```

기존에 같은 api 데이터에서 각각 props를 내려 받던 여러 개의 컴포넌트를 OrdeSummary라는 Context 하위에 존재하도록 만들었습니다. 주문자 페이지인지 배송자 페이지인지 구별할 수 있도록
isOrderer라는 추가 플래그를 같이 넘겨줍니다.

```jsx
const OrderCard = () =>{
  const { title, isOrderer, status } = useOrderSummaryContext();
  const reprocessTtitle = isOrderer ? title : `주문 명: ${title}`;
  const infoMessageMap = {
    [OrderStatus.OrderCreated]: '주문이 생성되었습니다.',
    [OrderStatus.MatchingCompleted]: '배송자와 매칭되었습니다.'
    {...}
  }

const
  return(
    <Box className="OrderCard">
      <SummaryTitle title={reprocessTtitle}>
      <Typography> {infoMessageMap[status]}</Typography>
      {isOrderer ? (<IsBoxIcon>) : null}
    </Box>
  )
}

```

context에서 필요한 데이터만 받아 뷰만 보여주는 SummaryTitle 컴포넌트에 title 주문자인지 배송자인지 가공 처리하여 넘겨줍니다. 얼핏 보면 전보다 훨신 깔끔해보이는데요..? 추가 요구 사항이 생겼습니다.

> 기획자 🙋🏻‍♀️: status가 ooo상태일 땐 Card 영역과 Contents의 영역이 순서가 바뀌고 Button을 미 노출 하게 해주세요. 또 Card 영역 안에 버튼이 3개 정도 생길 거예요~

으음... 뭔가 또 OrderCard에서 처리하다 보면 지난 실수를 반복할 것 같습니다.

### 2. 두번째 시도

지난 프로젝트엔 status가 6개였는데 이번엔 12개로 늘었습니다. 또 주문자 배송자 나뉘면 총 24개의 각기 다른 화면이 필요하네요...한 컴포넌트에서 24개의 분기 처리를 하는것은 좋지 않을 것 같아 과감히 OrderCard라는 컴포넌트를 버리겠습니다.

```jsx
cosnt OrderSummaryPage = () => {
const {data} = useGetOrderDetail();

const statusComponent = (status) => {
  switch(status) {
    case OrderStatus.OrderCreated:
    return <OrderSummary.OrdererOrderCreated />;
    case OrderStatus.MatchingCompleted:
    return <OrderSummary.OrdererMatchingCompleted />;
    //( 상태에 따른 나열 ... )
  }
}

  return(
    <div>
      <Header>
      <OrderSummary {...data} isOrderer={true}>
        {statusComponent(data?.status)}
      </OrderSummary>
      <FooterButton/>
    </div>
  )
}
```

```jsx
// 주문자의 status가 OrderCreated일때의 컴포넌트
cosnt OrderOrderCreated = () => {
  const { title } = useOrderSummaryContext();

  return(
    <>
     <OrderCardContainer>
        <SummaryTitle title={title}>
        <BoxIcon >
      </OrderCardContainer>
      {/* components...*/}
      <SummaryButton />
    </>
  )
}

// 베송자의 status가 Matching일때의 컴포넌트
cosnt OrderOrderCreated = () => {
  const { title } = useOrderSummaryContext();

  return(
    <>
      <OrderCardContainer>
        <SummaryTitle title={`주문 명: ${title}`}>
         {/* components...*/}
      </OrderCardContainer>
     {/* components...*/}
     <SummaryButton />
    </>
  )
}
```

기존에 UI를 기준으로 Context 하위 컴포넌트를 나누었다면 이번엔 크게 주문자, 배송자 크게 2벌로 나뉘고 그 안에 status별로 또 상태를 나누었습니다. 기존의 OrderCard를 해체하고 layout만을 제공하는 OrderCardContainer를 제작하여 컴포넌트 간 자율성을 부여했습니다. 비록 파일량은 전보다 10배는 늘었지만 같은 ui를 재사용할 수 있게 되엇고 또 코드 추적이 편해졌습니다.

```bash
├── OrderSummary
│   ├── status
│   │   ├── orderder
│   │   │   ├── OrderCreated.tsx
│   │   │   ├── MatchingCompleted.tsx
│   │   │   ├── PickupCompleted.tsx
│   │   │   ├── DeliveryCompleted.tsx
│   │   │   ├── OrderComfirm.tsx
│   │   │   └── OrderCanceled.tsx
│   │   └── carrier
│   │       ├── OrderCreated.tsx
│   │       ├── MatchingCompleted.tsx
│   │       ├── PickupCompleted.tsx
│   │       ├── DeliveryCompleted.tsx
│   │       ├── OrderComfirm.tsx
│   │       └── OrderCanceled.tsx
│   ├── OrderSummary.tsx
│   └── OrderSummary.scss

```

(실제로는 파일이 이것보다 더 많다)

<br/>
하지만 Context가 비대하게 커졌다는 느낌이 들고 아무래도 관리하는 파일이 너무 많아지는게 조금 마음에 걸립니다.. 과연 성공적인 리팩토링이 맞을까요?

## 3. 리팩토링 성공의 판단 기준?

가장 빠르고 쉬운 방법은 다른 개발자들에게 코드 리뷰를 받는 것입니다. 아무래도 내가 짠 코드는 객관적으로 바라보기가 힘드니까요. 하지만 코드리뷰를 받을 수 없는 환경이라면 어떻게 객관적으로 판단할 수 있을까요? 물론 개인별로 중요시하는 포인트를 우선적으로 판단을 하겠지만 저는 그 포인트를 무엇으로 잡아야할지 가늠조차 되지 않았습니다.

<br/>

> 🤔 뭔가.. 리팩토링 후 재활용성이 더 좋아지긴 했는데...? 과연 이게 좋은걸까?(나를 향한 불신)
> <br/>

계속 이에 대한 고민을 가지고 있었는데 멘토님들에게 여쭤볼 기회가 생겨 물어봤습니다.

![refactoring1.png](refactoring1.png)
</br>
이에 대해 다양한 답변을 주셨는데 궁금증이 많이 해소가 되었습니다!

> 💁‍♀️: 리팩토링 후 다음 날 자고 일어났을 때 내가 그 코드를 잘 이해한다면 성공  
> 💁🏻‍♂️: 기능이 추가되고 수정되었을 때 대응이 빨라지는가?  
> 💁: ai한테 물어보는 방법도 괜찮습니다.

<br/>

![refactoring2.png](refactoring2.png)

<br/>

저는 일단 현 프로젝트에서는 기능 대응에 초점을 두기로 했어요. 처음부터 구조를 잘 짜면 좋겠지만 아직은 초기 설계가 힘드네요. 또다시 리팩토링을 하게 된다면 의존성이 너무 강해지지 않을 정도로만 합성 컴포넌트 안에 다른 합성 컴포넌트를 넣어서 설계해 볼까 생각도 듭니다!.

```toc

```
