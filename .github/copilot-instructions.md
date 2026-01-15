# AI Coding Agent Instructions for discourse-discord-bot

## Project Overview
This is a Discourse plugin that integrates a Discord bot to synchronize messages, roles, and announcements between Discord and Discourse forums. It uses the `discordrb` gem for Discord API interactions and hooks into Discourse's event system.

## Architecture
- **Core Components**: `lib/bot.rb` (main bot class), `lib/bot_commands.rb` (admin commands), `lib/discord_events_handlers.rb` (Discord→Discourse sync), `lib/discourse_events_handlers.rb` (Discourse→Discord announcements), `lib/utils.rb` (message processing utilities)
- **Configuration**: `config/settings.yml` defines site settings; translations in `config/locales/`
- **Integration**: Plugin loads via `plugin.rb`, runs as a threaded bot within Discourse's Rails environment

## Key Patterns
- **Commands**: Prefixed with `!` (e.g., `!disccopy`, `!disckick`, `!discsync`), require admin role via `required_roles: [SiteSetting.discord_bot_admin_role_id]`
- **Rate Limiting**: Use `bucket` for command throttling, `sleep(SiteSetting.discord_bot_rate_limit_delay)` between API calls
- **Multi-tenancy**: Wrap logic in `RailsMultisite::ConnectionManagement.each_connection` to handle multiple Discourse instances
- **User Mapping**: Link Discord users via `UserAssociatedAccount` with provider 'discord'; fallback to proxy account for unknowns
- **Message Processing**: Use `::DiscordBot::Utils.prepare_post()` to convert Discord messages to Discourse posts, handling embeds, attachments, timestamps, and mentions
- **Event Handling**: Discord messages auto-sync to matching category topics; Discourse posts/topics announce to configured channels

## Development Workflow
- **Testing**: Run Discourse tests with `bundle exec rake`; plugin tests integrate with Discourse's test suite
- **Debugging**: Check Discourse logs for bot errors; use `Rails.logger.error` for custom logging
- **Settings Access**: Always use `SiteSetting.discord_bot_*` for configuration; changes trigger bot restart via `DiscourseEvent.on(:site_setting_changed)`
- **Threading**: Bot runs in background thread; handle exceptions to prevent crashes

## Common Tasks
- **Adding Commands**: Extend `manage_discord_commands()` in `bot_commands.rb` with rate limiting and role checks
- **Event Hooks**: Add to `DiscordEventsHandlers` modules for new Discord events; use `DiscourseEvent.on()` for Discourse events
- **Message Formatting**: Update `utils.rb` for new Discord content types (e.g., embeds, attachments)
- **Localization**: Add keys to `config/locales/server.*.yml` and use `I18n.t()` for user-facing strings

## Examples
- Sync Discord channel to Discourse: `!disccopy 50 category_name topic_name`
- Auto-sync: Enable `discord_bot_auto_channel_sync` to mirror channels by name
- Role sync: `!discsync` pulls Discourse groups as Discord roles for linked users