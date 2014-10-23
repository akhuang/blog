jquery ajax默认为异步请求, 但在实际开发过程中却发现在请求过程中会阻塞其它请求, 当前请求完成后才会处理其它请求, 这不符合异步请求的场景, 问题在哪呢?

Http请求：get / post

get从服务器端请求数据, 会自动缓存本地, 不安全; post不会自动缓存

使用$.get/$.getJSON异步请求时, 需要设置cache:false, 否则为同步请求, 没有异步效果;

Controller:

		private static readonly Random _random = new Random();

        public ActionResult Ajax()
        {
            var startTime = DateTime.Now;
            Thread.Sleep(_random.Next(5000, 10000));
            return Json(new
            {
                startTime = startTime.ToString("HH:mm:ss fff"),
                endTime = DateTime.Now.ToString("HH:mm:ss fff")
            }, JsonRequestBehavior.AllowGet);
        }


如下代码不会异步效果

      for (var i = 0; i < 6; i++) {
                    $.getJSON('/home/ajax', function (result) {
                        $('#result').append($('<div/>').html(
                            result.startTime + ' | ' + result.endTime
                        ));
                    });
                }
       } 

如果将代码改成

                for (var i = 0; i < 6; i++) {
                    $.ajax({
                        url: '/home/ajax',
                        cache: false,
                        success: function (result) {
                            $('#result').append($('<div/>').html(
                                result.startTime + ' | ' + result.endTime
                            ));
                        }
                    });
                };

关键在<code>cache: false</code>

但在MVC中还有一种情况也会导致异步失效, 那就是使用了Session的时候

###[ASP.NET Session State Overview](http://msdn.microsoft.com/en-us/library/ms178581.aspx)
	
> Access to ASP.NET session state is exclusive per session, which means that if two different users make concurrent requests, access to each separate session is granted concurrently. However, if two concurrent requests are made for the same session (by using the same SessionID value), the first request gets exclusive access to the session information. The second request executes only after the first request is finished.

所以要避免访问Session


参考:

1. [Why would multiple simultaneous AJAX calls to the same ASP.NET MVC action cause the browser to block?](http://stackoverflow.com/questions/4428413/why-would-multiple-simultaneous-ajax-calls-to-the-same-asp-net-mvc-action-cause)
2. [ASP.NET Session State Overview](http://msdn.microsoft.com/en-us/library/ms178581.aspx)
3. [Asynchronous Controller is blocking requests in ASP.NET MVC through jQuery](http://stackoverflow.com/questions/2927726/asynchronous-controller-is-blocking-requests-in-asp-net-mvc-through-jquery)
