# Nextのチュートリアル実践
    Next.jsのチュートリアルを進めているのでそのアウトプットとして学んだことを記述する
## CSSスタイリング
### グローバルスタイル・・・アプリケーション内すべてのルートにCSSルールを追加できる
/app/page.tsx
```tsx
import '@/app/ui/global.css';
export default function RootLayout({
children,
}: {
children: React.ReactNode;
}) {
return (
    <html lang="en">
    <body>{children}</body>
    </html>
);
}
```
### CSSモジュール・・・一意のクラス名を自動的に生成できるので衝突を防げる
/app/ui/home.module.css
```css
.shape {
    height: 0;
    width: 0;
    border-bottom: 30px solid black;
    border-left: 20px solid transparent;
    border-right: 20px solid transparent;
}
```

/app/page.tsx
```tsx
import styles from '@/app/ui/home.module.css';
<div className={styles.shape} />;
```

### clsx・・・状態や条件によって要素のスタイルを切り替える
/app/ui/invoices/status.tsx
```tsx
import clsx from 'clsx';
 
export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```

## レイアウトとページの作成
### ネストされたルーディング
Next.jsではフォルダーを使用してルートを作成するファイルシステムルーディングを使用する。<br>
例: URL: http://localhost:3000/dashboard/invoices
```
- project
  -app
    -page.tsx
    -layout.tsx
    -dashboard
        -invoices //こいつ
        -customers
```

## ページ間の移動
### ナビゲーションを最適化する理由
aタグのhrefを使うとページ全体がリロードされてしまい、パフォーマンスが低下する。Linkコンポーネントを使用する必要がある。

### コンポーネント<Link>
アプリケーション全体をレンダリングすることなく一部を更新できるようになる。<br>
/app/ui/dashboard/nav-links.tsx
```tsx
import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
 
// ...
 
export default function NavLinks() {
  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
```

### パターン: アクティブ リンクの表示
現在どのページのURLかを取得する必要がある。これはNext.jsのusePathname()というフックを使ってパスをチェックできる。<br />
※フックを使用する場合クライアントコンポーネントに変換する必要がある。<br />
/app/ui/dashboard/nav-links.tsx
```tsx
'use client'; //こいつを宣言してクライアントコンポーネントに変換する
 
import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import clsx from 'clsx';
 
// ...
 
export default function NavLinks() {
  const pathname = usePathname();
 
  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className={clsx(
              'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
              {
                'bg-sky-100 text-blue-600': pathname === link.href,
              },
            )}
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
 
// ...
```

## データの取得
### データの取得方法
#### APIレイヤー
- APIを提供しているサードパーティサービスを使用する場合
- クライアントからデータを取得する場合
#### データベースクエリ
- APIエンドポイントを作成する場合
- サーバーコンポーネントを使用している場合

### ダッシュボードの概要ページのデータの取得
例: RevenueChartコンポーネントのデータ取得する場合<br>
/app/dashboard/page.tsx
```tsx
import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';
import { fetchRevenue } from '@/app/lib/data'; // データ取得のコンポーネント
 
export default async function Page() {
    const revenue = await fetchRevenue(); //非同期で呼び出し
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        {/* <Card title="Collected" value={totalPaidInvoices} type="collected" /> */}
        {/* <Card title="Pending" value={totalPendingInvoices} type="pending" /> */}
        {/* <Card title="Total Invoices" value={numberOfInvoices} type="invoices" /> */}
        {/* <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        /> */}
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <RevenueChart revenue={revenue}  />
        {/* <LatestInvoices latestInvoices={latestInvoices} /> */}
      </div>
    </main>
  );
}
```

### リクエストウォーターフォールとは?
データ取得の処理が複数個存在する場合、前のデータ取得が完了に依存する一連のネットワーク要求。
例:
sample.ts
```ts
const res1 = await fetch("url1");// 2秒
const res2 = await fetch("url2");// 1秒
//res1の行の処理が完了してからres2の行の処理が完了する、3秒かかってしまう
```

### 並列データ取得
Promise.all()などを使用して並行に処理する
例:
sample.ts
```ts
const res1 = fetch("url1");// 2秒
const res2 = fetch("url2");// 1秒

const [res1, res2] = await Promise.all([promise1, promise2]);
//合計で2秒
```

## 静的レンダリングと動的レンダリング
### 静的レンダリングとは
データの取得とレンダリングをビルド時にサーバー上で行われる。
メリット
 - webサイトの高速化
 - サーバーの負荷の軽減

### ダイナミックレンダリングとは
クライアントからのリクエスト時にサーバー上でレンダリングされる。
メリット
 - リアルタイムでデータを取得
 - ユーザーの操作に基づいてデータの更新することが容易
 - リクエスト時に認識できる情報にアクセスできる。cookieやURLパラメータ

### ダッシュボードを動的にする
サーバーコンポーネント内でunstable_noStoreを使い静的レンダリングをオプトアウトする<br>
/app/lib/data.ts
```tsx
// ...
import { unstable_noStore as noStore } from 'next/cache';
 
export async function fetchRevenue() {
  // Add noStore() here to prevent the response from being cached.
  // This is equivalent to in fetch(..., {cache: 'no-store'}).
  noStore();
 
  // ...
}
 
export async function fetchLatestInvoices() {
  noStore();
  // ...
}
 
export async function fetchCardData() {
  noStore();
  // ...
}
 
export async function fetchFilteredInvoices(
  query: string,
  currentPage: number,
) {
  noStore();
  // ...
}
 
export async function fetchInvoicesPages(query: string) {
  noStore();
  // ...
}
 
export async function fetchFilteredCustomers(query: string) {
  noStore();
  // ...
}
 
export async function fetchInvoiceById(query: string) {
  noStore();
  // ...
}
```

## ストリーミング
### ストリーミングとは
ストリーミングは、ルートを小さな「チャンク」に分割し、準備ができたらサーバーからクライアントに段階的にストリーミングできるようにするデータ転送手法です。<br>
ストリーミングすることでページの要求が遅いときに全体でロードが完了するまで待つのではなく個々にページを表示することができる。<br>

### ページ全体をストリーミングする
/app/dashboard/loading.tsx
```tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

### ローディングスケルトンの追加
ローディングが完了するまでの間UIが簡略化されたようなものを表示する。<br>
/app/dashboard/loading.tsx
```tsx
import DashboardSkeleton from '@/app/ui/skeletons';
 
export default function Loading() {
  return <DashboardSkeleton />;
}
```

### コンポーネントのストリーミング
今の状態ではページ全体をストリーミングしている。ページの中のコンポーネントごとでローディングが終わるごとに個々に表示したい場合、Suspenseコンポーネントを使用する。<br>
/app/dashboard/(概要)/page.tsx
```tsx
import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';
import { fetchLatestInvoices, fetchCardData } from '@/app/lib/data';
import { Suspense } from 'react';
import { RevenueChartSkeleton } from '@/app/ui/skeletons';
 
export default async function Page() {
  const latestInvoices = await fetchLatestInvoices();
  const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();
 
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        <Card title="Collected" value={totalPaidInvoices} type="collected" />
        <Card title="Pending" value={totalPendingInvoices} type="pending" />
        <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
        <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        />
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <Suspense fallback={<RevenueChartSkeleton />}>
          <RevenueChart />
        </Suspense>
        <LatestInvoices latestInvoices={latestInvoices} />
      </div>
    </main>
  );
}
```

/app/ui/dashboard/revenue-chart.tsx
```tsx
import { generateYAxis } from '@/app/lib/utils';
import { CalendarIcon } from '@heroicons/react/24/outline';
import { lusitana } from '@/app/ui/fonts';
import { fetchRevenue } from '@/app/lib/data';
 
// ...
 
export default async function RevenueChart() { // Make component async, remove the props
  const revenue = await fetchRevenue(); // Fetch data inside the component
 
  const chartHeight = 350;
  const { yAxisLabels, topLabel } = generateYAxis(revenue);
 
  if (!revenue || revenue.length === 0) {
    return <p className="mt-4 text-gray-400">No data available.</p>;
  }
 
  return (
    // ...
  );
}
```