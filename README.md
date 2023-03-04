# Typecho-Tips: Realize the link in the article to open in a new tab / Typecho技巧：实现文章内链接在新标签打开
在拿到一个没有经过魔改，或者使用未考虑链接跳转这种情况的主题时，该文章就会对你很有用处。  默认情况Typecho文章中如果有添加链接，那么是从当前窗口跳转的，并且外链没有添加nofollow标签，不利于SEO，Typecho文章内链接新窗口并添加nofollow标签如下。


# 方法一：修改Typecho系统文件

## Typecho 1.0 之前的老版本

###### 找到 /var/CommonMark/HtmlRenderer.php 这个文件，在105行，也就是 $attrs['href'] = $this->escape($inline->getAttribute('destination'), true); 代码之后添加如下两行代码：


    $attrs['target'] = $this->escape(_blank, true);
    $attrs['rel'] = $this->escape(nofollow, true);

###### 最后保存查看效果，修改后的完整代码，请注意对比：


    case CommonMark_Element_InlineElement::TYPE_LINK:
                    $attrs['href'] = $this->escape($inline->getAttribute('destination'), true);
                    $attrs['target'] = $this->escape(_blank, true);
                    $attrs['rel'] = $this->escape(nofollow, true);
                    if ($title = $inline->getAttribute('title')) {
                        $attrs['title'] = $this->escape($title, true);
                    }
		
## Typecho 1.0 以上新版本

######  修改Typecho系统代码

###### 找到 varMarkdown.php 这个文件，这里和上面老版本文件不同。

在 self::$parser->hook('afterParseCode', array('Markdown', 'transerCodeClass')); 后面，添加下面代码


    self::$parser->hook('afterParseInline', array('Markdown', 'addLinkTargetBlank'));

###### 然后在 public static function transerCodeClass($html){} 这个代码块后面，添加下面代码


    /**
     * addLinkTargetBlank
     * 
     * @param string $html
     * @return string
     */
    public static function addLinkTargetBlank($html)
    {
        return preg_replace("/<a href=\"([^\"]*)\">/i", "<a href=\"\\1\" target=\"_blank\">", $html);
    }

###### 如果需要添加rel=nofollow，则如下


    /**
     * addLinkTargetBlank
     * 
     * @param string $html
     * @return string
     */
    public static function addLinkTargetBlank($html)
    {
        return preg_replace("/<a href=\"([^\"]*)\">/i", "<a href=\"\\1\" target=\"_blank\" rel=\"nofollow\">", $html);
    }

# 方法二：前端通过JS跳转

###### 【本人建议：不推荐，兼容性差，容易报错】


###### 在前端,通过Vanilla JS解决


###### 逻辑是查找某个标签的id下所有的a标签，然后给每个添加属性，如下


    var pageAnchors = document.getElementById('post-content').getElementsByTagName('a');
    for (var i=0; i<pageAnchors.length; i++){
        pageAnchors[i].setAttribute('target', '_blank');
	}
    
###### 这样的是不可靠，在没有某个标签的id的情况下浏览器会报错，一报错，我就不舒服，解决如下
    

    if(document.getElementById('post-content')){
        var pageAnchors = document.getElementById('post-content').getElementsByTagName('a');
        for (var i=0; i<pageAnchors.length; i++){
            pageAnchors[i].setAttribute('target', '_blank');
        }
    }

###### 套个方法，美化一下，如下


    var pageTagretBlank = function(pageId) {
       if(document.getElementById(pageId)){
            var pageAnchors = document.getElementById(pageId).getElementsByTagName('a');
            for (var i=0; i<pageAnchors.length; i++){
                pageAnchors[i].setAttribute('target', '_blank');
            }
        }
    };
    pageTagretBlank('post-content');

###### 如果我还需要添加rel=nofollow，则如下


    var pageTagretBlank = function(pageId) {
        if(document.getElementById(pageId)){
            var pageAnchors = document.getElementById(pageId).getElementsByTagName('a');
            for (var i=0; i<pageAnchors.length; i++){
                pageAnchors[i].setAttribute('target', '_blank');
                postAnchors[i].setAttribute('rel', 'nofollow');
            }
        }
    };
    pageTagretBlank('post-content');

###### 代码一般放在主题的footer.php就可以了

# 方法三：通过PHP函数实现

    【本人建议：推荐，兼容性好】
    
###### 直接在主题里集成文章链接新窗口跳转，在function.php的添加 parseContent() 函数，函数为


    //未添加nofollow
     function parseContent($obj){
        $options = Typecho_Widget::widget('Widget_Options');
        if(!empty($options->src_add) && !empty($options->cdn_add)){
            $obj->content = str_ireplace($options->src_add,$options->cdn_add,$obj->content);
        }
        $obj->content = preg_replace("/<a href=\"([^\"]*)\">/i", "<a href=\"\\1\" target=\"_blank\">", $obj->content);
        echo trim($obj->content);
    }

###### 如果需要添加rel=nofollow，则如下


    //添加nofollow
    function parseContent($obj){
        $options = Typecho_Widget::widget('Widget_Options');
        if(!empty($options->src_add) && !empty($options->cdn_add)){
            $obj->content = str_ireplace($options->src_add,$options->cdn_add,$obj->content);
        }
        $obj->content = preg_replace("/<a href=\"([^\"]*)\">/i", "<a href=\"\\1\" target=\"_blank\" rel=\"nofollow\">", $obj->content);
        echo trim($obj->content);
    }

###### 该方法的原理就是正则文章的超链接标签，然后加上相应处理即可。使用该方法需要修改主题 post.php 文件，将默认的内容输出

<?php $this->content(); ?> 改成 <?php parseContent($this); ?> 。

#方法四：通过修改主题Header.php文件

######【本人建议：不推荐，简单粗暴，不够灵活】
    
###### 最近看到一种新的方法是通过修改主题header.php文件在顶部加上<base target="_blank"/>即可。

![13](https://user-images.githubusercontent.com/43232263/222863599-4dce266d-22e2-4076-89ec-74986f013b04.png)

