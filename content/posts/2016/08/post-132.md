---
title: '[cocos2dx] IntのVectorクラスを雑に作った'
author: しゃまとん
type: post
date: 2016-08-29T13:01:33+00:00
url: /posts/132
featured_image: /images/posts/2016/08/0027_3.jpg
categories:
  - cocos2dx
  - プログラミング関連

---
お世話になっております。  
しゃまとんです。

以前にcocosで作業をしていた際に、タイルマップ的な処理をするときに通常のVectorクラスはfloat型（だったかな）で定義されているため、Int型で使いたいと思うと毎回キャストしないといけません。

そこで自作でInt型のVectorクラスを作って利用していました。少し前のものですが、そんなに特殊なことはしていないので使えると思います（ます）。

もし同じように考えている方がいらっしゃれば、ご自由にお使いいただければ幸いです。  
（ビルド通らなかったらごめんなさい）

<pre class="lang:c++ decode:true brush: cpp; gutter: true" title="IntVec2.cpp">//
//  IntVec2.cpp
//

#include "IntVec2.h"


IntVec2::IntVec2()
: x(0), y(0)
{
}

IntVec2::IntVec2(int xx, int yy)
: x(xx), y(yy)
{
}

IntVec2::IntVec2(const IntVec2& p1, const IntVec2& p2)
{
    set(p1, p2);
}

IntVec2::IntVec2(const IntVec2& copy)
{
    set(copy);
}

IntVec2::~IntVec2()
{
}


void IntVec2::set(int xx, int yy)
{
    this-&gt;x = xx;
    this-&gt;y = yy;
}

void IntVec2::set(const IntVec2& v)
{
    this-&gt;x = v.x;
    this-&gt;y = v.y;
}

void IntVec2::set(const IntVec2& p1, const IntVec2& p2)
{
    x = p2.x - p1.x;
    y = p2.y - p1.y;
}


void IntVec2::setPoint(int xx, int yy)
{
    this-&gt;x = xx;
    this-&gt;y = yy;
}

bool IntVec2::equals(const IntVec2& target) const
{
    return ((this-&gt;x == target.x) && (this-&gt;y == target.y));
}

void IntVec2::negate()
{
    x = -x;
    y = -y;
}

void IntVec2::add(const IntVec2& v)
{
    x += v.x;
    y += v.y;
}

void IntVec2::subtract(const IntVec2& v)
{
    x -= v.x;
    y -= v.y;
}

void IntVec2::scale(int scalar)
{
    x *= scalar;
    y *= scalar;
}

void IntVec2::scale(const IntVec2& scale)
{
    x *= scale.x;
    y *= scale.y;
}

//---------------------------------------------------------
// oprator
//---------------------------------------------------------
const IntVec2 IntVec2::operator+(const IntVec2& v) const
{
    IntVec2 result(*this);
    result.add(v);
    return result;
}

IntVec2& IntVec2::operator+=(const IntVec2& v)
{
    add(v);
    return *this;
}

const IntVec2 IntVec2::operator-(const IntVec2& v) const
{
    IntVec2 result(*this);
    result.subtract(v);
    return result;
}

IntVec2& IntVec2::operator-=(const IntVec2& v)
{
    subtract(v);
    return *this;
}

const IntVec2 IntVec2::operator-() const
{
    IntVec2 result(*this);
    result.negate();
    return result;
}

const IntVec2 IntVec2::operator*(int s) const
{
    IntVec2 result(*this);
    result.scale(s);
    return result;
}

IntVec2& IntVec2::operator*=(int s)
{
    scale(s);
    return *this;
}

const IntVec2 IntVec2::operator/(const int s) const
{
    return IntVec2(this-&gt;x / s, this-&gt;y / s);
}

bool IntVec2::operator&lt;(const IntVec2& v) const
{
    if (x == v.x)
    {
        return y &lt; v.y;
    }
    return x &lt; v.x;
}

bool IntVec2::operator==(const IntVec2& v) const
{
    return x == v.x && y == v.y;
}

bool IntVec2::operator!=(const IntVec2& v) const
{
    return x != v.x || y != v.y;
}

const IntVec2 operator*(float x, const IntVec2& v)
{
    IntVec2 result(v);
    result.scale(x);
    return result;
}</pre>

&nbsp;

<pre class="lang:c++ decode:true brush: cpp; gutter: true" title="IntVec2.h">//
//  IntVec2.h
//

#ifndef __IntVec2__
#define __IntVec2__

#include &lt;algorithm&gt;
#include &lt;functional&gt;
#include &lt;math.h&gt;
#include "math/CCMathBase.h"

/**
 * Defines a 2-element int point vector.
 */
class IntVec2
{
public:

    /**
     * The x coordinate.
     */
    int x;

    /**
     * The y coordinate.
     */
    int y;

    IntVec2();
    IntVec2(int xx, int yy);
    IntVec2(const IntVec2& p1, const IntVec2& p2);
    IntVec2(const IntVec2& copy);

    /**
     * Destructor.
     */
    ~IntVec2();

    void set(int xx, int yy);
    void set(const int* array);
    void set(const IntVec2& v);
    void set(const IntVec2& p1, const IntVec2& p2);

    void negate();

    void add(const IntVec2& v);

    void scale(int scalar);
    void scale(const IntVec2& scale);

    void subtract(const IntVec2& v);


    const IntVec2 operator+(const IntVec2& v) const;
    IntVec2& operator+=(const IntVec2& v);
    const IntVec2 operator-(const IntVec2& v) const;
    IntVec2& operator-=(const IntVec2& v);
    const IntVec2 operator-() const;
    const IntVec2 operator*(int s) const;
    IntVec2& operator*=(int s);
    const IntVec2 operator/(int s) const;
    bool operator&lt;(const IntVec2& v) const;
    bool operator==(const IntVec2& v) const;
    bool operator!=(const IntVec2& v) const;

    //code added compatible for Point
public:
    /**
     * @js NA
     * @lua NA
     */
    void setPoint(int xx, int yy);
    /**
     * @js NA
     */
    bool equals(const IntVec2& target) const;

};


#endif /* defined(__IntVec2__) */</pre>

以上です。