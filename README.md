{"name": "justsaju",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"},
  "dependencies": {
    "next": "14.0.0",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "openai": "^4.0.0"
  }
}
import { useState, useEffect } from "react";

export default function Home() {
  const [question, setQuestion] = useState("");
  const [result, setResult] = useState("");
  const [paid, setPaid] = useState(false);

  useEffect(() => {
    if (localStorage.getItem("paid") === "true") {
      setPaid(true);
    }
  }, []);

  const handlePayment = async () => {
    const res = await fetch("/api/kakao-pay-ready", {
      method: "POST",
    });
    const data = await res.json();
    window.location.href = data.next_redirect_pc_url;
  };

  const handleSubmit = async () => {
    if (!paid) return alert("결제를 먼저 해주세요");

    setResult("AI가 사주 분석 중입니다...");

    const res = await fetch("/api/saju", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ question }),
    });

    const data = await res.json();
    setResult(data.result);
  };

  return (
    <div style={{ padding: 40 }}>
      <h1>저스트사주</h1>

      <button onClick={handlePayment}>
        카카오페이 결제 (4,000원)
      </button>

      <br /><br />

      <textarea
        value={question}
        onChange={(e) => setQuestion(e.target.value)}
        placeholder="생년월일 + 질문 입력"
        style={{ width: "100%", height: 100 }}
      />

      <br /><br />

      <button onClick={handleSubmit}>
        사주 보기
      </button>

      <p style={{ marginTop: 20, whiteSpace: "pre-line" }}>
        {result}
      </p>
    </div>
  );
}
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export default async function handler(req, res) {
  const { question } = req.body;

  const completion = await openai.chat.completions.create({
    model: "gpt-4o-mini",
    messages: [
      {
        role: "system",
        content: "너는 20년 경력의 사주 전문가다. 현실적으로 구체적으로 말해라.",
      },
      {
        role: "user",
        content: question,
      },
    ],
  });

  res.status(200).json({
    result:
      completion.choices[0].message.content +
      "\n\n👉 더 깊은 분석은 심층 사주를 추천드립니다",
  });
}
export default async function handler(req, res) {
  const response = await fetch("https://kapi.kakao.com/v1/payment/ready", {
    method: "POST",
    headers: {
      Authorization: `KakaoAK ${process.env.KAKAO_ADMIN_KEY}`,
      "Content-type": "application/x-www-form-urlencoded;charset=utf-8",
    },
    body: new URLSearchParams({
      cid: "TC0ONETIME",
      partner_order_id: "order_" + Date.now(),
      partner_user_id: "user_1",
      item_name: "사주 1질문",
      quantity: "1",
      total_amount: "4000",
      vat_amount: "0",
      tax_free_amount: "0",
      approval_url: "https://너의주소/success",
      cancel_url: "https://너의주소/cancel",
      fail_url: "https://너의주소/fail",
    }),
  });

  const data = await response.json();
  res.status(200).json(data);
}
import { useEffect } from "react";
import { useRouter } from "next/router";

export default function Success() {
  const router = useRouter();

  useEffect(() => {
    localStorage.setItem("paid", "true");
    alert("결제 완료!");
    router.push("/");
  }, []);

  return <div>결제 처리중...</div>;
}
OPENAI_API_KEY=여기에키
KAKAO_ADMIN_KEY=여기에키
