It is a lightweight, simple, low memory usage, non-verifying HTML stream parser.
Originally written for embedded devices.
Parsing HTML stream char by char.
Not building HTML tree.
Supports invalid HTML code.

Here are a simple C programs that illustrates how to use the parser API.

You can run all programs with this command (assuming a name of the program is "a.out"):
```
wget -O - -q "http://code.google.com" | ./a.out
```

This one prints a tags names not using buffer.
```
#include <stdio.h>
#include <htmlstreamparser.h>

int main(void) {

	char c, new_line = 1;

	HTMLSTREAMPARSER *hsp = html_parser_init();

	while ((c = getchar()) != EOF) {

		html_parser_char_parse(hsp, c);

		if (html_parser_is_in(hsp, HTML_NAME)) {
			new_line = 1;
			printf("%c", c);
		} else if (new_line) {
			new_line = 0;
			printf("\n");
		}
	}

	html_parser_cleanup(hsp);
}
```
This one prints a tags names using buffer.
```
#include <stdio.h>
#include <htmlstreamparser.h>

int main(void) {

	char c, tag[11];
	size_t tag_len;

	HTMLSTREAMPARSER *hsp = html_parser_init();

	html_parser_set_tag_buffer(hsp, tag, sizeof(tag)-1);

	while ((c = getchar()) != EOF) {
		html_parser_char_parse(hsp, c);
		if (html_parser_is_in(hsp, HTML_TAG_END)) {
			tag_len = html_parser_tag_length(hsp);
			tag[tag_len] = '\0';
			printf("%s\n", tag);
		}
	}
	html_parser_cleanup(hsp);
}
```
This one prints a web page title.
```
#include <stdio.h>
#include <htmlstreamparser.h>

int main(void) {

	char c, tag[6], *title, inner[128];
	size_t title_len = 0;

	HTMLSTREAMPARSER *hsp = html_parser_init();

	html_parser_set_tag_to_lower(hsp, 1);

	html_parser_set_tag_buffer(hsp, tag, sizeof(tag));
	html_parser_set_inner_text_buffer(hsp, inner, sizeof(inner)-1);

	while ((c = getchar()) != EOF) {
		html_parser_char_parse(hsp, c);
		if (html_parser_cmp_tag(hsp, "/title", 6)) {
			title_len = html_parser_inner_text_length(hsp);
			title = html_parser_replace_spaces(html_parser_trim(html_parser_inner_text(hsp), &title_len), &title_len);
			break; // or html_parser_release_inner_text_buffer to continue
		}
		
	}
	html_parser_cleanup(hsp);

	if (title_len > 0) {
		title[title_len] = '\0';
		printf("%s<\n", title);
	} else {
		printf("no title\n");
	}
}
```
This one prints an attribute href value from the anchors tags.
```
#include <stdio.h>
#include <htmlstreamparser.h>

int main(void) {

	char c, tag[1], attr[4], val[128];

	HTMLSTREAMPARSER *hsp = html_parser_init();

	html_parser_set_tag_to_lower(hsp, 1);
	html_parser_set_attr_to_lower(hsp, 1);

	html_parser_set_tag_buffer(hsp, tag, sizeof(tag));
	html_parser_set_attr_buffer(hsp, attr, sizeof(attr));
	html_parser_set_val_buffer(hsp, val, sizeof(val)-1);

	while ((c = getchar()) != EOF) {
		html_parser_char_parse(hsp, c);
		if (html_parser_cmp_tag(hsp, "a", 1))
			if (html_parser_cmp_attr(hsp, "href", 4))
				if (html_parser_is_in(hsp, HTML_VALUE_ENDED)) {
					val[html_parser_val_length(hsp)] = '\0';
					printf("%s\n", val);
				}
	}
	html_parser_cleanup(hsp);
}
```
This one prints actually parsed char with HTML part description.
```
#include <stdio.h>
#include <htmlstreamparser.h>

void print_html_part(HTMLSTREAMPARSER *hsp) {
	if (html_parser_is_in(hsp, HTML_INNER_TEXT)) printf(" HTML_INNER_TEXT");
	if (html_parser_is_in(hsp, HTML_TAG)) printf(" HTML_TAG");
	if (html_parser_is_in(hsp, HTML_TAG_BEGINNING)) printf(" HTML_TAG_BEGINNING");
	if (html_parser_is_in(hsp, HTML_TAG_END)) printf(" HTML_TAG_END");
	if (html_parser_is_in(hsp, HTML_NAME)) printf(" HTML_NAME");
	if (html_parser_is_in(hsp, HTML_NAME_BEGINNING)) printf(" HTML_NAME_BEGINNING");
	if (html_parser_is_in(hsp, HTML_NAME_ENDED)) printf(" HTML_NAME_ENDED");
	if (html_parser_is_in(hsp, HTML_ATTRIBUTE)) printf(" HTML_ATTRIBUTE");
	if (html_parser_is_in(hsp, HTML_ATTRIBUTE_BEGINNING)) printf(" HTML_ATTRIBUTE_BEGINNING");
	if (html_parser_is_in(hsp, HTML_ATTRIBUTE_ENDED)) printf(" HTML_ATTRIBUTE_ENDED");
	if (html_parser_is_in(hsp, HTML_VALUE)) printf(" HTML_VALUE");
	if (html_parser_is_in(hsp, HTML_VALUE_BEGINNING)) printf(" HTML_VALUE_BEGINNING");
	if (html_parser_is_in(hsp, HTML_VALUE_ENDED)) printf(" HTML_VALUE_ENDED");
	if (html_parser_is_in(hsp, HTML_VALUE_QUOTED)) printf(" HTML_VALUE_QUOTED");
	if (html_parser_is_in(hsp, HTML_VALUE_SINGLE_QUOTED)) printf(" HTML_VALUE_SINGLE_QUOTED");
	if (html_parser_is_in(hsp, HTML_VALUE_DOUBLE_QUOTED)) printf(" HTML_VALUE_DOUBLE_QUOTED");
	if (html_parser_is_in(hsp, HTML_SPACE)) printf(" HTML_SPACE");
	if (html_parser_is_in(hsp, HTML_EQUALITY)) printf(" HTML_EQUALITY");
	if (html_parser_is_in(hsp, HTML_SLASH)) printf(" HTML_SLASH");
	if (html_parser_is_in(hsp, HTML_CLOSING_TAG)) printf(" HTML_CLOSING_TAG");
	if (html_parser_is_in(hsp, HTML_SCRIPT)) printf(" HTML_SCRIPT");
	if (html_parser_is_in(hsp, HTML_COMMENT)) printf(" HTML_COMMENT");
	if (html_parser_is_in(hsp, HTML_ENTITY)) printf(" HTML_ENTITY");
}

int main(void) {
	char c;
	HTMLSTREAMPARSER *hsp = html_parser_init();
	while ((c = getchar()) != EOF) {
		html_parser_char_parse(hsp, c);
		switch (c) {
			case '\t': printf("\\t"); break;
			case '\n': printf("\\n"); break;
			case '\r': printf("\\r"); break;
			default: printf(" %c", c);
		}
		print_html_part(hsp);
		printf("\n");
	}
	html_parser_cleanup(hsp);
}
```