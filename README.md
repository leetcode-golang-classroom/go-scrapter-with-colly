# go-scrapter-with-colly

This is a example with web scraping with colly framework

## install colly

```shell
go get -u github.com/gocolly/colly/...
```

## write logic to scraing the content from website
Define structure for element
```golang
type item struct {
	Name   string `json:"name"`
	Price  string `json:"price"`
	ImgUrl string `json:"imgurl"`
}
```
First visit setup the collector

```golang
c := colly.NewCollector(
		colly.AllowedDomains("j2store.net"), // this is for target domain
	)
```

Then, Setup the selector for target element, callback for scraping

```golang
var items []item
	c.OnHTML("div.col-sm-9 div[itemprop=itemListElement]", func(h *colly.HTMLElement) {
		item := item{
			Name:   h.ChildText("h2.product-title"),
			Price:  h.ChildText("div.sale-price"),
			ImgUrl: h.ChildAttr("img", "src"),
		}
		items = append(items, item)
	})
```

With Selector, we could use element to send url for visit page

```golang
c.OnHTML("[title=Next]", func(h *colly.HTMLElement) {
		next_page := h.Request.AbsoluteURL(h.Attr("href"))
		c.Visit(next_page)
	})
```

Then use visit to actually visit page
```golang
c.Visit("http://j2store.net/demo/index.php/shop")
	content, err := json.Marshal(items)
	if err != nil {
		fmt.Println(err.Error())
	}
```

## callback for send request from callback

```golang
c.OnRequest(func(r *colly.Request) {
		fmt.Println(r.URL.String())
	})
```

