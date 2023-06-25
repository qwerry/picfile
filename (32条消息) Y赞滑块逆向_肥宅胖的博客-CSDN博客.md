# (32条消息) Y赞滑块逆向_肥宅胖的博客-CSDN博客
好久没写文章了，最近刚转正比较忙,今天有空把业务中写过的一个滑块来给大家讲讲吧。  
话不多说直接上网站：aHR0cHMlM0EvL2FjY291bnQueW91emFuLmNvbS9sb2dpbg==

直接开搞：

第一步;[抓包](https://so.csdn.net/so/search?q=%E6%8A%93%E5%8C%85&spm=1001.2101.3001.7020)分析：  
![](https://img-blog.csdnimg.cn/2020081116443935.png)
  
我们多刷新抓几次包分析可以得到下面这些结论：

**token：统一请求标识  
bizType：固定  
bizData：  
captchaType:固定  
userBechaviorData：轨迹加密****

我们发现token是前面返回的那么整个接口[加密](https://so.csdn.net/so/search?q=%E5%8A%A0%E5%AF%86&spm=1001.2101.3001.7020)就剩一个userBehaviorData了  
那就简单了啊 我们直接全局搜索：  
![](https://img-blog.csdnimg.cn/20200811165028258.png)
  
找到位置打上断点，在滑一次滑块。  
在控制台输出一下加密的值：  
![](https://img-blog.csdnimg.cn/20200811165125836.png)
  
然后输出一下加密后的结果：  
![](https://img-blog.csdnimg.cn/20200811165159302.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTI3OTk1,size_16,color_FFFFFF,t_70)
  
果然是这个地方，那我们只需要构造出来加密参数，和扣出加密算法就行了。  
来  
喽他！  
加密参数的分析：  
“{“cx”:230,“cy”:43,“scale”:0.5,“slidingEvents”:\[{“mx”:40,“my”:211,“ts”:1595474135296},{“mx”:1,“my”:0,“ts”:39},{“mx”:3,“my”:0,“ts”:8},{“mx”:5,“my”:0,“ts”:9},{“mx”:5,“my”:0,“ts”:7},{“mx”:7,“my”:0,“ts”:9},{“mx”:9,“my”:-1,“ts”:7},{“mx”:14,“my”:-1,“ts”:9},{“mx”:14,“my”:-1,“ts”:7},{“mx”:14,“my”:-2,“ts”:9},{“mx”:14,“my”:-2,“ts”:7},{“mx”:17,“my”:-1,“ts”:9},{“mx”:20,“my”:-1,“ts”:8},{“mx”:20,“my”:-1,“ts”:8},{“mx”:24,“my”:0,“ts”:8},{“mx”:20,“my”:-2,“ts”:8},{“mx”:20,“my”:-1,“ts”:8},{“mx”:17,“my”:-1,“ts”:8},{“mx”:13,“my”:0,“ts”:8},{“mx”:11,“my”:0,“ts”:8},{“mx”:15,“my”:0,“ts”:7},{“mx”:11,“my”:0,“ts”:8},{“mx”:9,“my”:0,“ts”:8},{“mx”:6,“my”:0,“ts”:8},{“mx”:3,“my”:1,“ts”:10},{“mx”:3,“my”:0,“ts”:6},{“mx”:1,“my”:0,“ts”:8}\]}”  
我们观察到他的json格式都为x,y,t 格式  
“cx”:230,“cy”:43,“scale”:0.5  
前面这里我们猜测cx=230为滑块缺口，cy=43为鼠标点击高度，scale=0.5为滑动时间。  
\[{“mx”:40,“my”:211,“ts”:1595474135296}  
后面这一部分json我们猜测是在一段时间是收集x和y和t的移动距离  
ts在\[6,7,8,9,10\]这个区间里，那么我们只需要将全部x坐标加起来等于230，就知道他是不是这样检测的。  
我们发现结果不等于，所以我们暂时不知道他是检测什么的遇到这种情况我们直接写死试试，最后发现果然  
他自己都不会检测这个轨迹。  
所以我们直接保存一条轨迹出来，替换里面的时间戳和终点坐标就行。

```
def distcance(juli):
    x='{"cx":'+str(int((juli)/2))+',"cy":18,"scale":0.5,"slidingEvents":[{"mx":40,"my":196,"ts":'+str(int(time.time()*1000))+'}' \
     ',{"mx":1,"my":0,"ts":1},{"mx":1,"my":-1,"ts":8},{"mx":1,"my":0,"ts":8},{"mx":4,"my":0,"ts":9},{"mx":3,"my":-1,"ts":7},' \
      '{"mx":7,"my":-1,"ts":8},{"mx":12,"my":-1,"ts":7},{"mx":12,"my":-1,"ts":9},{"mx":19,"my":-2,"ts":7},{"mx":22,"my":-1,"ts":9},' \
     '{"mx":31,"my":-2,"ts":7},{"mx":33,"my":-1,"ts":8},{"mx":35,"my":-2,"ts":8},{"mx":38,"my":0,"ts":8},{"mx":42,"my":-2,"ts":8},' \
       '{"mx":43,"my":0,"ts":8},{"mx":43,"my":0,"ts":8},{"mx":44,"my":0,"ts":8},{"mx":46,"my":0,"ts":8},{"mx":38,"my":0,"ts":8},' \
        '{"mx":38,"my":0,"ts":8},{"mx":33,"my":-1,"ts":8},{"mx":25,"my":-2,"ts":8},{"mx":19,"my":0,"ts":8},{"mx":15,"my":0,"ts":8},' \
      '{"mx":9,"my":-1,"ts":8},{"mx":11,"my":0,"ts":8},{"mx":4,"my":-1,"ts":8},{"mx":6,"my":0,"ts":8},{"mx":1,"my":0,"ts":8}]}'
    with open(r'get_data.js', 'r', encoding='UTF-8')as f:
        ua_js = f.read().encode().decode("gbk", 'ignore')
    js_data = execjs.compile(ua_js)
    key = js_data.call('get', x)
    return key

```

参数分析完了 ，我们来分析加密函数：  
逆向userBechaviorData算法  
为对称加密AES，采用cbc填充方式，iv为偏移值  
我们可用python进行改写，也可以直接扣js  
这里我们采用直接扣去js的方法  
![](https://img-blog.csdnimg.cn/20200811165444696.png)
  
我们跟进去发现：  
他所有的方法都在这个里面

![](https://img-blog.csdnimg.cn/2020081116553940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTI3OTk1,size_16,color_FFFFFF,t_70)
  
但是我们直接抠下来并不能直接用，因为他被

```
void 0 === (r = function(t, e, n) {})

```

包着的我们需要自己稍微改写下：

```
var wGdk = function(t, e, n) {
    var r;
    var r, o, i = i || function(t, e) {
            var n = {}, r = n.lib = {}, o = function() {}, i = r.Base = {
                extend: function(t) {
                    o.prototype = this;
                    var e = new o;
                    return t && e.mixIn(t), e.hasOwnProperty("init") || (e.init = function() {
                        e.$super.init.apply(this, arguments)
                    }), e.init.prototype = e, e.$super = this, e
                },
                create: function() {
                    var t = this.extend();
                    return t.init.apply(t, arguments), t
                },
                init: function() {},
                mixIn: function(t) {
                    for (var e in t)
                    t.hasOwnProperty(e) && (this[e] = t[e]);
                    t.hasOwnProperty("toString") && (this.toString = t.toString)
                },
                clone: function() {
                    return this.init.prototype.extend(this)
                }
            }, u = r.WordArray = i.extend({
                init: function(t, e) {
                    t = this.words = t || [], this.sigBytes = null != e ? e : 4 * t.length
                },
                toString: function(t) {
                    return (t || c).stringify(this)
                },
                concat: function(t) {
                    var e = this.words,
                        n = t.words,
                        r = this.sigBytes;
                    if (t = t.sigBytes, this.clamp(), r % 4) for (var o = 0; o < t; o++)
                    e[r + o >>> 2] |= (n[o >>> 2] >>> 24 - o % 4 * 8 & 255) << 24 - (r + o) % 4 * 8;
                    else if (65535 < n.length) for (o = 0; o < t; o += 4)
                    e[r + o >>> 2] = n[o >>> 2];
                    else e.push.apply(e, n);
                    return this.sigBytes += t, this
                },
                clamp: function() {
                    var e = this.words,
                        n = this.sigBytes;
                    e[n >>> 2] &= 4294967295 << 32 - n % 4 * 8, e.length = t.ceil(n / 4)
                },
                clone: function() {
                    var t = i.clone.call(this);
                    return t.words = this.words.slice(0), t
                },
                random: function(e) {
                    for (var n = [], r = 0; r < e; r += 4)
                    n.push(4294967296 * t.random() | 0);
                    return new u.init(n, e)
                }
            }),
                a = n.enc = {}, c = a.Hex = {
                    stringify: function(t) {
                        var e = t.words;
                        t = t.sigBytes;
                        for (var n = [], r = 0; r < t; r++) {
                            var o = e[r >>> 2] >>> 24 - r % 4 * 8 & 255;
                            n.push((o >>> 4).toString(16)), n.push((15 & o).toString(16))
                        }
                        return n.join("")
                    },
                    parse: function(t) {
                        for (var e = t.length, n = [], r = 0; r < e; r += 2)
                        n[r >>> 3] |= parseInt(t.substr(r, 2), 16) << 24 - r % 8 * 4;
                        return new u.init(n, e / 2)
                    }
                }, s = a.Latin1 = {
                    stringify: function(t) {
                        var e = t.words;
                        t = t.sigBytes;
                        for (var n = [], r = 0; r < t; r++)
                        n.push(String.fromCharCode(e[r >>> 2] >>> 24 - r % 4 * 8 & 255));
                        return n.join("")
                    },
                    parse: function(t) {
                        for (var e = t.length, n = [], r = 0; r < e; r++)
                        n[r >>> 2] |= (255 & t.charCodeAt(r)) << 24 - r % 4 * 8;
                        return new u.init(n, e)
                    }
                }, f = a.Utf8 = {
                    stringify: function(t) {
                        try {
                            return decodeURIComponent(escape(s.stringify(t)))
                        } catch (t) {
                            throw Error("Malformed UTF-8 data")
                        }
                    },
                    parse: function(t) {
                        return s.parse(unescape(encodeURIComponent(t)))
                    }
                }, l = r.BufferedBlockAlgorithm = i.extend({
                    reset: function() {
                        this._data = new u.init, this._nDataBytes = 0
                    },
                    _append: function(t) {
                        "string" == typeof t && (t = f.parse(t)), this._data.concat(t), this._nDataBytes += t.sigBytes
                    },
                    _process: function(e) {
                        var n = this._data,
                            r = n.words,
                            o = n.sigBytes,
                            i = this.blockSize,
                            a = o / (4 * i);
                        if (e = (a = e ? t.ceil(a) : t.max((0 | a) - this._minBufferSize, 0)) * i, o = t.min(4 * e, o), e) {
                            for (var c = 0; c < e; c += i)
                            this._doProcessBlock(r, c);
                            c = r.splice(0, e), n.sigBytes -= o
                        }
                        return new u.init(c, o)
                    },
                    clone: function() {
                        var t = i.clone.call(this);
                        return t._data = this._data.clone(), t
                    },
                    _minBufferSize: 0
                });
            r.Hasher = l.extend({
                cfg: i.extend(),
                init: function(t) {
                    this.cfg = this.cfg.extend(t), this.reset()
                },
                reset: function() {
                    l.reset.call(this), this._doReset()
                },
                update: function(t) {
                    return this._append(t), this._process(), this
                },
                finalize: function(t) {
                    return t && this._append(t), this._doFinalize()
                },
                blockSize: 16,
                _createHelper: function(t) {
                    return function(e, n) {
                        return new t.init(n).finalize(e)
                    }
                },
                _createHmacHelper: function(t) {
                    return function(e, n) {
                        return new p.HMAC.init(t, n).finalize(e)
                    }
                }
            });
            var p = n.algo = {};
            return n
        }(Math);
    o = (r = i).lib.WordArray, r.enc.Base64 = {
        stringify: function(t) {
            var e = t.words,
                n = t.sigBytes,
                r = this._map;
            t.clamp(), t = [];
            for (var o = 0; o < n; o += 3)
            for (var i = (e[o >>> 2] >>> 24 - o % 4 * 8 & 255) << 16 | (e[o + 1 >>> 2] >>> 24 - (o + 1) % 4 * 8 & 255) << 8 | e[o + 2 >>> 2] >>> 24 - (o + 2) % 4 * 8 & 255, u = 0; 4 > u && o + .75 * u < n; u++)
            t.push(r.charAt(i >>> 6 * (3 - u) & 63));
            if (e = r.charAt(64)) for (; t.length % 4;)
            t.push(e);
            return t.join("")
        },
        parse: function(t) {
            var e = t.length,
                n = this._map;
            (r = n.charAt(64)) && -1 != (r = t.indexOf(r)) && (e = r);
            for (var r = [], i = 0, u = 0; u < e; u++)
            if (u % 4) {
                var a = n.indexOf(t.charAt(u - 1)) << u % 4 * 2,
                    c = n.indexOf(t.charAt(u)) >>> 6 - u % 4 * 2;
                r[i >>> 2] |= (a | c) << 24 - i % 4 * 8, i++
            }
            return o.create(r, i)
        },
        _map: "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="
    },
    function(t) {
        function e(t, e, n, r, o, i, u) {
            return ((t = t + (e & n | ~e & r) + o + u) << i | t >>> 32 - i) + e
        }

        function n(t, e, n, r, o, i, u) {
            return ((t = t + (e & r | n & ~r) + o + u) << i | t >>> 32 - i) + e
        }

        function r(t, e, n, r, o, i, u) {
            return ((t = t + (e ^ n ^ r) + o + u) << i | t >>> 32 - i) + e
        }

        function o(t, e, n, r, o, i, u) {
            return ((t = t + (n ^ (e | ~r)) + o + u) << i | t >>> 32 - i) + e
        }
        for (var u = i, a = (s = u.lib).WordArray, c = s.Hasher, s = u.algo, f = [], l = 0; 64 > l; l++)
        f[l] = 4294967296 * t.abs(t.sin(l + 1)) | 0;
        s = s.MD5 = c.extend({
            _doReset: function() {
                this._hash = new a.init([1732584193, 4023233417, 2562383102, 271733878])
            },
            _doProcessBlock: function(t, i) {
                for (var u = 0; 16 > u; u++) {
                    var a = t[c = i + u];
                    t[c] = 16711935 & (a << 8 | a >>> 24) | 4278255360 & (a << 24 | a >>> 8)
                }
                u = this._hash.words;
                var c = t[i + 0],
                    s = (a = t[i + 1], t[i + 2]),
                    l = t[i + 3],
                    p = t[i + 4],
                    h = t[i + 5],
                    d = t[i + 6],
                    v = t[i + 7],
                    g = t[i + 8],
                    y = t[i + 9],
                    m = t[i + 10],
                    b = t[i + 11],
                    x = t[i + 12],
                    w = t[i + 13],
                    _ = t[i + 14],
                    S = t[i + 15],
                    O = e(O = u[0], j = u[1], D = u[2], P = u[3], c, 7, f[0]),
                    P = e(P, O, j, D, a, 12, f[1]),
                    D = e(D, P, O, j, s, 17, f[2]),
                    j = e(j, D, P, O, l, 22, f[3]);
                O = e(O, j, D, P, p, 7, f[4]), P = e(P, O, j, D, h, 12, f[5]), D = e(D, P, O, j, d, 17, f[6]), j = e(j, D, P, O, v, 22, f[7]), O = e(O, j, D, P, g, 7, f[8]), P = e(P, O, j, D, y, 12, f[9]), D = e(D, P, O, j, m, 17, f[10]), j = e(j, D, P, O, b, 22, f[11]), O = e(O, j, D, P, x, 7, f[12]), P = e(P, O, j, D, w, 12, f[13]), D = e(D, P, O, j, _, 17, f[14]), O = n(O, j = e(j, D, P, O, S, 22, f[15]), D, P, a, 5, f[16]), P = n(P, O, j, D, d, 9, f[17]), D = n(D, P, O, j, b, 14, f[18]), j = n(j, D, P, O, c, 20, f[19]), O = n(O, j, D, P, h, 5, f[20]), P = n(P, O, j, D, m, 9, f[21]), D = n(D, P, O, j, S, 14, f[22]), j = n(j, D, P, O, p, 20, f[23]), O = n(O, j, D, P, y, 5, f[24]), P = n(P, O, j, D, _, 9, f[25]), D = n(D, P, O, j, l, 14, f[26]), j = n(j, D, P, O, g, 20, f[27]), O = n(O, j, D, P, w, 5, f[28]), P = n(P, O, j, D, s, 9, f[29]), D = n(D, P, O, j, v, 14, f[30]), O = r(O, j = n(j, D, P, O, x, 20, f[31]), D, P, h, 4, f[32]), P = r(P, O, j, D, g, 11, f[33]), D = r(D, P, O, j, b, 16, f[34]), j = r(j, D, P, O, _, 23, f[35]), O = r(O, j, D, P, a, 4, f[36]), P = r(P, O, j, D, p, 11, f[37]), D = r(D, P, O, j, v, 16, f[38]), j = r(j, D, P, O, m, 23, f[39]), O = r(O, j, D, P, w, 4, f[40]), P = r(P, O, j, D, c, 11, f[41]), D = r(D, P, O, j, l, 16, f[42]), j = r(j, D, P, O, d, 23, f[43]), O = r(O, j, D, P, y, 4, f[44]), P = r(P, O, j, D, x, 11, f[45]), D = r(D, P, O, j, S, 16, f[46]), O = o(O, j = r(j, D, P, O, s, 23, f[47]), D, P, c, 6, f[48]), P = o(P, O, j, D, v, 10, f[49]), D = o(D, P, O, j, _, 15, f[50]), j = o(j, D, P, O, h, 21, f[51]), O = o(O, j, D, P, x, 6, f[52]), P = o(P, O, j, D, l, 10, f[53]), D = o(D, P, O, j, m, 15, f[54]), j = o(j, D, P, O, a, 21, f[55]), O = o(O, j, D, P, g, 6, f[56]), P = o(P, O, j, D, S, 10, f[57]), D = o(D, P, O, j, d, 15, f[58]), j = o(j, D, P, O, w, 21, f[59]), O = o(O, j, D, P, p, 6, f[60]), P = o(P, O, j, D, b, 10, f[61]), D = o(D, P, O, j, s, 15, f[62]), j = o(j, D, P, O, y, 21, f[63]);
                u[0] = u[0] + O | 0, u[1] = u[1] + j | 0, u[2] = u[2] + D | 0, u[3] = u[3] + P | 0
            },
            _doFinalize: function() {
                var e = this._data,
                    n = e.words,
                    r = 8 * this._nDataBytes,
                    o = 8 * e.sigBytes;
                n[o >>> 5] |= 128 << 24 - o % 32;
                var i = t.floor(r / 4294967296);
                for (n[15 + (o + 64 >>> 9 << 4)] = 16711935 & (i << 8 | i >>> 24) | 4278255360 & (i << 24 | i >>> 8), n[14 + (o + 64 >>> 9 << 4)] = 16711935 & (r << 8 | r >>> 24) | 4278255360 & (r << 24 | r >>> 8), e.sigBytes = 4 * (n.length + 1), this._process(), n = (e = this._hash).words, r = 0; 4 > r; r++)
                o = n[r], n[r] = 16711935 & (o << 8 | o >>> 24) | 4278255360 & (o << 24 | o >>> 8);
                return e
            },
            clone: function() {
                var t = c.clone.call(this);
                return t._hash = this._hash.clone(), t
            }
        }), u.MD5 = c._createHelper(s), u.HmacMD5 = c._createHmacHelper(s)
    }(Math),
    function() {
        var t, e = i,
            n = (t = e.lib).Base,
            r = t.WordArray,
            o = (t = e.algo).EvpKDF = n.extend({
                cfg: n.extend({
                    keySize: 4,
                    hasher: t.MD5,
                    iterations: 1
                }),
                init: function(t) {
                    this.cfg = this.cfg.extend(t)
                },
                compute: function(t, e) {
                    for (var n = (a = this.cfg).hasher.create(), o = r.create(), i = o.words, u = a.keySize, a = a.iterations; i.length < u;) {
                        c && n.update(c);
                        var c = n.update(t).finalize(e);
                        n.reset();
                        for (var s = 1; s < a; s++)
                        c = n.finalize(c), n.reset();
                        o.concat(c)
                    }
                    return o.sigBytes = 4 * u, o
                }
            });
        e.EvpKDF = function(t, e, n) {
            return o.create(n).compute(t, e)
        }
    }(), i.lib.Cipher || function(t) {
        var e = (d = i).lib,
            n = e.Base,
            r = e.WordArray,
            o = e.BufferedBlockAlgorithm,
            u = d.enc.Base64,
            a = d.algo.EvpKDF,
            c = e.Cipher = o.extend({
                cfg: n.extend(),
                createEncryptor: function(t, e) {
                    return this.create(this._ENC_XFORM_MODE, t, e)
                },
                createDecryptor: function(t, e) {
                    return this.create(this._DEC_XFORM_MODE, t, e)
                },
                init: function(t, e, n) {
                    this.cfg = this.cfg.extend(n), this._xformMode = t, this._key = e, this.reset()
                },
                reset: function() {
                    o.reset.call(this), this._doReset()
                },
                process: function(t) {
                    return this._append(t), this._process()
                },
                finalize: function(t) {
                    return t && this._append(t), this._doFinalize()
                },
                keySize: 4,
                ivSize: 4,
                _ENC_XFORM_MODE: 1,
                _DEC_XFORM_MODE: 2,
                _createHelper: function(t) {
                    return {
                        encrypt: function(e, n, r) {
                            return ("string" == typeof n ? v : h).encrypt(t, e, n, r)
                        },
                        decrypt: function(e, n, r) {
                            return ("string" == typeof n ? v : h).decrypt(t, e, n, r)
                        }
                    }
                }
            });
        e.StreamCipher = c.extend({
            _doFinalize: function() {
                return this._process(!0)
            },
            blockSize: 1
        });
        var s = d.mode = {}, f = function(t, e, n) {
            var r = this._iv;
            r ? this._iv = void 0 : r = this._prevBlock;
            for (var o = 0; o < n; o++)
            t[e + o] ^= r[o]
        }, l = (e.BlockCipherMode = n.extend({
            createEncryptor: function(t, e) {
                return this.Encryptor.create(t, e)
            },
            createDecryptor: function(t, e) {
                return this.Decryptor.create(t, e)
            },
            init: function(t, e) {
                this._cipher = t, this._iv = e
            }
        })).extend();
        l.Encryptor = l.extend({
            processBlock: function(t, e) {
                var n = this._cipher,
                    r = n.blockSize;
                f.call(this, t, e, r), n.encryptBlock(t, e), this._prevBlock = t.slice(e, e + r)
            }
        }), l.Decryptor = l.extend({
            processBlock: function(t, e) {
                var n = this._cipher,
                    r = n.blockSize,
                    o = t.slice(e, e + r);
                n.decryptBlock(t, e), f.call(this, t, e, r), this._prevBlock = o
            }
        }), s = s.CBC = l, l = (d.pad = {}).Pkcs7 = {
            pad: function(t, e) {
                for (var n, o = (n = (n = 4 * e) - t.sigBytes % n) << 24 | n << 16 | n << 8 | n, i = [], u = 0; u < n; u += 4)
                i.push(o);
                n = r.create(i, n), t.concat(n)
            },
            unpad: function(t) {
                t.sigBytes -= 255 & t.words[t.sigBytes - 1 >>> 2]
            }
        }, e.BlockCipher = c.extend({
            cfg: c.cfg.extend({
                mode: s,
                padding: l
            }),
            reset: function() {
                c.reset.call(this);
                var t = (e = this.cfg).iv,
                    e = e.mode;
                if (this._xformMode == this._ENC_XFORM_MODE) var n = e.createEncryptor;
                else n = e.createDecryptor, this._minBufferSize = 1;
                this._mode = n.call(e, this, t && t.words)
            },
            _doProcessBlock: function(t, e) {
                this._mode.processBlock(t, e)
            },
            _doFinalize: function() {
                var t = this.cfg.padding;
                if (this._xformMode == this._ENC_XFORM_MODE) {
                    t.pad(this._data, this.blockSize);
                    var e = this._process(!0)
                } else e = this._process(!0), t.unpad(e);
                return e
            },
            blockSize: 4
        });
        var p = e.CipherParams = n.extend({
            init: function(t) {
                this.mixIn(t)
            },
            toString: function(t) {
                return (t || this.formatter).stringify(this)
            }
        }),
            h = (s = (d.format = {}).OpenSSL = {
                stringify: function(t) {
                    var e = t.ciphertext;
                    return ((t = t.salt) ? r.create([1398893684, 1701076831]).concat(t).concat(e) : e).toString(u)
                },
                parse: function(t) {
                    var e = (t = u.parse(t)).words;
                    if (1398893684 == e[0] && 1701076831 == e[1]) {
                        var n = r.create(e.slice(2, 4));
                        e.splice(0, 4), t.sigBytes -= 16
                    }
                    return p.create({
                        ciphertext: t,
                        salt: n
                    })
                }
            }, e.SerializableCipher = n.extend({
                cfg: n.extend({
                    format: s
                }),
                encrypt: function(t, e, n, r) {
                    r = this.cfg.extend(r);
                    var o = t.createEncryptor(n, r);
                    return e = o.finalize(e), o = o.cfg, p.create({
                        ciphertext: e,
                        key: n,
                        iv: o.iv,
                        algorithm: t,
                        mode: o.mode,
                        padding: o.padding,
                        blockSize: t.blockSize,
                        formatter: r.format
                    })
                },
                decrypt: function(t, e, n, r) {
                    return r = this.cfg.extend(r), e = this._parse(e, r.format), t.createDecryptor(n, r).finalize(e.ciphertext)
                },
                _parse: function(t, e) {
                    return "string" == typeof t ? e.parse(t, this) : t
                }
            })),
            d = (d.kdf = {}).OpenSSL = {
                execute: function(t, e, n, o) {
                    return o || (o = r.random(8)), t = a.create({
                        keySize: e + n
                    }).compute(t, o), n = r.create(t.words.slice(e), 4 * n), t.sigBytes = 4 * e, p.create({
                        key: t,
                        iv: n,
                        salt: o
                    })
                }
            }, v = e.PasswordBasedCipher = h.extend({
                cfg: h.cfg.extend({
                    kdf: d
                }),
                encrypt: function(t, e, n, r) {
                    return n = (r = this.cfg.extend(r)).kdf.execute(n, t.keySize, t.ivSize), r.iv = n.iv, (t = h.encrypt.call(this, t, e, n.key, r)).mixIn(n), t
                },
                decrypt: function(t, e, n, r) {
                    return r = this.cfg.extend(r), e = this._parse(e, r.format), n = r.kdf.execute(n, t.keySize, t.ivSize, e.salt), r.iv = n.iv, h.decrypt.call(this, t, e, n.key, r)
                }
            })
    }(),
    function() {
        for (var t = i, e = t.lib.BlockCipher, n = t.algo, r = [], o = [], u = [], a = [], c = [], s = [], f = [], l = [], p = [], h = [], d = [], v = 0; 256 > v; v++)
        d[v] = 128 > v ? v << 1 : v << 1 ^ 283;
        var g = 0,
            y = 0;
        for (v = 0; 256 > v; v++) {
            var m = (m = y ^ y << 1 ^ y << 2 ^ y << 3 ^ y << 4) >>> 8 ^ 255 & m ^ 99;
            r[g] = m, o[m] = g;
            var b = d[g],
                x = d[b],
                w = d[x],
                _ = 257 * d[m] ^ 16843008 * m;
            u[g] = _ << 24 | _ >>> 8, a[g] = _ << 16 | _ >>> 16, c[g] = _ << 8 | _ >>> 24, s[g] = _, _ = 16843009 * w ^ 65537 * x ^ 257 * b ^ 16843008 * g, f[m] = _ << 24 | _ >>> 8, l[m] = _ << 16 | _ >>> 16, p[m] = _ << 8 | _ >>> 24, h[m] = _, g ? (g = b ^ d[d[d[w ^ b]]], y ^= d[d[y]]) : g = y = 1
        }
        var S = [0, 1, 2, 4, 8, 16, 32, 64, 128, 27, 54];
        n = n.AES = e.extend({
            _doReset: function() {
                for (var t = (n = this._key).words, e = n.sigBytes / 4, n = 4 * ((this._nRounds = e + 6) + 1), o = this._keySchedule = [], i = 0; i < n; i++)
                if (i < e) o[i] = t[i];
                else {
                    var u = o[i - 1];
                    i % e ? 6 < e && 4 == i % e && (u = r[u >>> 24] << 24 | r[u >>> 16 & 255] << 16 | r[u >>> 8 & 255] << 8 | r[255 & u]) : (u = r[(u = u << 8 | u >>> 24) >>> 24] << 24 | r[u >>> 16 & 255] << 16 | r[u >>> 8 & 255] << 8 | r[255 & u], u ^= S[i / e | 0] << 24), o[i] = o[i - e] ^ u
                }
                for (t = this._invKeySchedule = [], e = 0; e < n; e++)
                i = n - e, u = e % 4 ? o[i] : o[i - 4], t[e] = 4 > e || 4 >= i ? u : f[r[u >>> 24]] ^ l[r[u >>> 16 & 255]] ^ p[r[u >>> 8 & 255]] ^ h[r[255 & u]]
            },
            encryptBlock: function(t, e) {
                this._doCryptBlock(t, e, this._keySchedule, u, a, c, s, r)
            },
            decryptBlock: function(t, e) {
                var n = t[e + 1];
                t[e + 1] = t[e + 3], t[e + 3] = n, this._doCryptBlock(t, e, this._invKeySchedule, f, l, p, h, o), n = t[e + 1], t[e + 1] = t[e + 3], t[e + 3] = n
            },
            _doCryptBlock: function(t, e, n, r, o, i, u, a) {
                for (var c = this._nRounds, s = t[e] ^ n[0], f = t[e + 1] ^ n[1], l = t[e + 2] ^ n[2], p = t[e + 3] ^ n[3], h = 4, d = 1; d < c; d++) {
                    var v = r[s >>> 24] ^ o[f >>> 16 & 255] ^ i[l >>> 8 & 255] ^ u[255 & p] ^ n[h++],
                        g = r[f >>> 24] ^ o[l >>> 16 & 255] ^ i[p >>> 8 & 255] ^ u[255 & s] ^ n[h++],
                        y = r[l >>> 24] ^ o[p >>> 16 & 255] ^ i[s >>> 8 & 255] ^ u[255 & f] ^ n[h++];
                    p = r[p >>> 24] ^ o[s >>> 16 & 255] ^ i[f >>> 8 & 255] ^ u[255 & l] ^ n[h++], s = v, f = g, l = y
                }
                v = (a[s >>> 24] << 24 | a[f >>> 16 & 255] << 16 | a[l >>> 8 & 255] << 8 | a[255 & p]) ^ n[h++], g = (a[f >>> 24] << 24 | a[l >>> 16 & 255] << 16 | a[p >>> 8 & 255] << 8 | a[255 & s]) ^ n[h++], y = (a[l >>> 24] << 24 | a[p >>> 16 & 255] << 16 | a[s >>> 8 & 255] << 8 | a[255 & f]) ^ n[h++], p = (a[p >>> 24] << 24 | a[s >>> 16 & 255] << 16 | a[f >>> 8 & 255] << 8 | a[255 & l]) ^ n[h++], t[e] = v, t[e + 1] = g, t[e + 2] = y, t[e + 3] = p
            },
            keySize: 8
        });
        t.AES = e._createHelper(n)
    }(), i.pad.Iso10126 = {
        pad: function(t, e) {
            var n = (n = 4 * e) - t.sigBytes % n;
            t.concat(i.lib.WordArray.random(n - 1)).concat(i.lib.WordArray.create([n << 24], 1))
        },
        unpad: function(t) {
            t.sigBytes -= 255 & t.words[t.sigBytes - 1 >>> 2]
        }
    }
    return i
}

    function get(t1) {
        var e = new wGdk,
            r = e.enc.Utf8.parse("youzan.com.aesiv"),
            o = e.enc.Utf8.parse("youzan.com._key_");
        var t = e.enc.Utf8.parse(t1)
        return e.AES.encrypt(t, o, {
            mode: e.mode.CBC,
            padding: e.pad.Iso10126,
            iv: r
        }).toString()
    }

```

图像识别的代码网上到处都是，这就不给了  
有什么疑问可以私聊我。qq:1374522338  
木木哒。