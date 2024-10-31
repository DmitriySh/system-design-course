Homework 02: design a REST API by functional requirements
=======
[Homework description](https://balun-team.yonote.ru/share/5a9ad957-fdfe-43ab-99a9-3b47e529f782/doc/domashnee-zadanie-2-Gd1QOvbtb0)

## System design "Social network for travelers"

![image](https://github.com/user-attachments/assets/2712c495-7088-4746-a2e3-df7fb2bdcffc)

### Functional requirements:

- users make a new posts with photos, small description and geolocation spot of travel;
- users can rate posts other travelers;
- users can comment posts other travelers;
- users can subscribe to other travelers (relations between influencers and followers);
- search popular places for traveling and viewing posts about them;
- viewing the feed of other travelers.

### REST API:

Use [Open API](https://www.openapis.org) - formal standard for describing HTTP APIs. More details in the file
`/api/rest_api.yml` you can view on the website https://editor.swagger.io

REST API endpoints:

- posts
    - `POST /api/posts`
    - `GET /api/posts/{postId}`
    - `PUT /api/posts/{postId}`
    - `DELETE /api/posts/{postId}`
    - `GET /api/posts/user-feeds?userId=id&limit=10&offset=0`
    - `GET /api/posts/popular-locations?coordinateX=1.0&coordinateY=1.5&radius=100&limit=10&offset=0`
- attachments
    - `POST /api/attachments?resourceId=3fa85f64-5717-4562-b3fc-2c963f66afa6&resourceType=POST&attachmentType=IMAGE`
    - `GET /api/posts/{postId}/attachments`
    - `DELETE /api/attachments/{attachmentId}`
- reactions
    - `POST /api/posts/{postId}/reactions`
    - `GET /api/posts/{postId}/reactions?limit=10&offset=0`
    - `DELETE /api/reactions/{reactionId}`
- comments
    - `POST /api/posts/{postId}/comments`
    - `GET /api/posts/{postId}/comments?limit=10&offset=0`
    - `PUT /api/comments/{commentId}`
    - `DELETE /api/comments/{commentId}`
- subscriptions
    - `GET /api/users/{userId}/subscriptions?limit=10&offset=0`
    - `POST /api/users/{userId}/subscriptions?followingUserId=4da85f64-2317-4562-b3gj-1s963f89afa6`
    - `DELETE /api/subscriptions/{subscriptionId}`
- users
    - `GET /api/users/{userId}`
    - `GET /api/users?userIds=3fa85f64-5717-4562-b3fc-2c963f66afa6&userIds=...`
 